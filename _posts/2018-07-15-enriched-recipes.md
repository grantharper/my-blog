---
layout: post
title:  "Enriched Recipes"
date:   2018-07-15
categories: software
---

Following recipes is the way a lot of people cook, especially those who like to explore new cuisine in a guided manner. A recipe book is a tried and true friend, but does it present information in a way that naturally flows with the process of cooking? Because of the limitations of the static format of a book, the text of a recipe is an inefficient representation of the cooking process.

### Typical Recipe Format

1. A list of the ingredients along with their quantities
  * 2 cups cherry tomatoes, halved
  * 5 1/2 tbsp olive oil, plus more for drizzling
  * 1 3/4 lbs white onions, cut into thin rings
  * 2 tbsp thyme leaves

2. A list of instructions to follow

  * Preheat the oven to 275F. Spread the tomatoes cut-side up or a small baking pan and sprinkle them with some salt and pepper and a drizzle of oil. Roast for 25 minutes, or until semi-cooked. They are not supposed to dry out completely. 
  * Meanwhile, heat up 4 tablespoons olive oil in a large frying pan: Add the onions, thyme and some salt and pepper and cook on high heat, stirring, for about a minute. Reduce the heat to low and continue cooking for 20 minutes, stirring occasionally...

The problem comes when the ingredients interface with the instructions. For example, let's say that our recipe says, "Add the flour." Your natural next question is, "How much?" To find that out, your eyes have to jump back up to the ingredients section and figure that out. If your recipe has a few ingredients, no problem, but you start to get dizzy after a while. 

### Initial Unenriched JSON

This is the format of the recipes that my recipe-ocr application outputs from scanned images.

```json
{
  "uri": "Plenty224",
  "book": "Plenty",
  "title": "Socca",
  "pageId": "224",
  "servingSize": "Serves 4",
  "ingredients": [
    "2 cups cherry tomatoes, halved",
    "5 1/2 tbsp olive oil, plus more for drizzling",
    "1 3/4 lbs white onions, cut into thin rings",
    "2 tbsp thyme leaves"
  ],
  "instructions": "Preheat the oven to 275F. Spread the tomatoes cut-side up or a small baking pan and sprinkle them with some salt and pepper and a drizzle of oil. Roast for 25 minutes, or until semi-cooked. They are not supposed to dry out completely. Meanwhile, heat up 4 tablespoons olive oil in a large frying pan: Add the onions, thyme and some salt and pepper and cook on high heat, stirring, for about a minute. Reduce the heat to low and continue cooking for 20 minutes, stirring occasionally..."
}


```

This works great for an elasticsearch-based search engine, but for the actual cooking process what we really want is an enhanced representation of the recipe that includes pointers from the instructions to the ingredients where they belong. Then whatever application using the information can make decisions about which representation makes sense.
1. Pure instructions with ingredients excluding amounts (ideal for a quick scan over a recipe)
2. List of ingredients (ideal for a shopping list)
3. Instructions with ingedient amounts included (ideal for the cooking process)

#### Enriched JSON

```json
{
  "uri": "Plenty224",
  "title": "Socca",
  "makes": "Serves 4",
  "time": "30 minutes",
  "access": "default",
  "attribution": "Plenty",
  "url": null,
  "ingredients": {
    "ingId0": {
      "text": "cherry tomatoes, halved",
      "quantity": "2 cups",
      "listOrder": 0
    },
    "ingId3": {
      "text": "thyme leaves",
      "quantity": "2 tbsp",
      "listOrder": 3
    },
    "ingId1": {
      "text": "olive oil, plus more for drizzling",
      "quantity": "5 1/2 tbsp",
      "listOrder": 1
    },
    "ingId2": {
      "text": "white onions, cut into thin rings",
      "quantity": "1 3/4 lbs",
      "listOrder": 2
    }
  },
  "steps": [
    "Preheat the oven to 275F. Spread the ${ingId0} cut-side up or a small baking pan and sprinkle them with some salt and pepper and a drizzle of oil.",
    "Roast for 25 minutes, or until semi-cooked. They are not supposed to dry out completely.",
    "Meanwhile, heat up 4 tablespoons ${ingId1} oil in a large frying pan: Add the ${ingId2}, ${ingId3} and some salt and pepper and cook on high heat, stirring, for about a minute."
  ]
}

```

### Enriching the Recipe

But how do we enrich the recipe? Looking at the first representation, we have to be able to perform the following main enrichments:

1. Split up ingredients from their quantities
2. Identify the ingredients within the instructions and replace them with variables that point back to the ingredients

Splitting the ingredients from their quantities is relatively straightforward and can be done with simple string operations. Identifying the ingredients is a little more tricky. 

### Apache Lucene to the rescue!

Using lucene's tokenization, we can tokenize the ingredient text, tokenize each step of the instructions, and then look through each step to identify where the ingredients belong. One of the beautiful things about lucene is its extensibility. 

In the code snippet below, I have added stopwords from an external file which I can easily modify as I discover additional words that would never be considered ingredients and should be completely ignored. By setting these stopwords, the tokenzier only creates tokens for words not contained in the list.

```java
ClassPathResource classPathResource = new ClassPathResource("recipe-stopwords.txt");
Set<String> stopwordsInFile = Files.readAllLines(Paths.get(classPathResource.getURI()))
                                    .stream()
                                    .map(String::trim)
                                    .collect(Collectors.toSet());
stopwordsInFile.addAll(standardStopWords);
CharArraySet stopWordsCharArray = new CharArraySet(stopwordsInFile, true);
analyzer = new EnglishAnalyzer(stopWordsCharArray);
```

In addition to stopwords, the anlayzer performs stemming so that things like plural words are reduced down so that they can be more easily compared.

For the recipe above, this what the tokenization looks like for step 0 and ingredient 0.

#### Ingredient 0

cherry tomatoes, halved 

<span>&rarr;</span> Analyzer <span>&rarr;</span> 

`[cherry, tomato]`

#### Step 0

Preheat the oven to 275F. Spread the tomatoes cut-side up or a small baking pan and sprinkle them with some salt and pepper and a drizzle of oil.

<span>&rarr;</span> Anlayzer <span>&rarr;</span>

`[preheat, oven, 275f, spread, tomato, cut, side, up, small, bake, pan, sprinkl, them, some, salt, pepper, drizzl, oil]`

Searching through these arrays, the system finds tomato as a match. Lucene has stored where the word that resulted in the token `tomato` was originally, and it can be replaced with `${ingId0}`

`Preheat the oven to 275F. Spread the ${ingId0} cut-side up or a small baking pan and sprinkle them with some salt and pepper and a drizzle of oil.`

### Processing the Templates

Finally, to get back to the different representations that we talked about above, we use Apache Freemarker to inject the ingredient info back into the instructions.

```java
String processTemplate(Map<String, String> ingredientMap, String unformattedStep)
  {
    try
    {
      StringWriter stringWriter = new StringWriter();
      Template template = new Template("step", new StringReader(unformattedStep), this.configuration);
      template.process(ingredientMap, stringWriter);
      return stringWriter.toString();
    }
    catch (TemplateException | IOException e)
    {
      e.printStackTrace();
      throw new RuntimeException("unable to process step: " + unformattedStep);
    }
  }
```

So now we have our three formats of step 0 and ingredient 0 of the recipe

1. Preheat the oven to 275F. Spread the tomatoes cut-side up or a small baking pan and sprinkle them with some salt and pepper and a drizzle of oil.
2. 2 cups cherry tomatoes, halved
3. Preheat the oven to 275F. Spread the 2 cups cherry tomatoes, halved cut-side up or a small baking pan and sprinkle them with some salt and pepper and a drizzle of oil.

Voila!