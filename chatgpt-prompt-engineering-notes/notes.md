# OpenAI - ChatGPT Prompt Engineering for Developers

## Two types of LLMs
1. Base LLM: Predicts next word based on text training data\
   Example:\
   a) **Once upon a time, there was a unicorn** _that lived in a magical forest with all her unicorn friends._ \
   b) **What is the capital of France?**\
      _What is France's largest city?_\
      _What is France's population?_\
      _What is the currency of France?_
   
   - A base LLM would not answer the question but rather complete the text that was given based on its training data. In this case, the data may be a list of exam questions.
     
2. Instruction Tuned LLM - tries to follow instructions\
   Example:\
   **What is the capital of France?**\
   _The capital of France is Paris._

   - Starts out with a base LLM that has been trained on massive amounts of text data and further fine-tune it with inputs and outputs that are instructions and good attempts to follow those instructions.
   - This is further refined using a technique called RLHF (Reinforced Learning with Human Feedback)
   - Mostly trained to be helpful, honest, and harmless so the output is less likely to produce toxic outputs compared to base LLMs

## Guidelines for prompting:
- Principle 1: Write clear and specific instructions. Clear does not equal short.
- Principle 2: Give the model time to think.

### Principle 1: Write clear and specific instructions
**Tactic 1:** 
- Use delimiters to clearly indicate distinct parts of the input. Eg: ''', """, <>, :, etc.
- Delimiters can be any kind of clear punctuation that separate specific pieces of text from the rest of the prompt. This makes it clear to the model that this is a separate section.
- Delimiters are also useful in preventing prompt injections.\
Example:
```python
   text = f"""
   Diamond rings are crafted through a detailed process. First, natural or lab-grown diamonds are cut and polished to enhance brilliance.
   A jeweler then designs a metal band—often gold, platinum, or silver—by melting and shaping it into the desired form. The diamond is securely
   set into the band using prongs, bezels, or other settings. After assembly, the ring is polished for a flawless finish. Final inspections
   ensure quality and durability before the ring is boxed and ready to shine.
   """
   prompt = f"""
   Summarize the following text delimited by <> into one sentence with less than 20 words.
   <{text}>
   """
```
   Output:
```
Diamond rings are meticulously crafted by cutting, polishing, setting, and inspecting before being boxed.
```

**Tactic 2:**
- Ask for a structured output like JSON, HTML, etc.\
Example:
```python
   prompt = f"""
   Generate a list of three baby names along with the gender, year of popularity, and how many percentage of the population has that name.
   Provide them in JSON format with the following keys: name, gender, year of popularity, percentage of population.
   """
```
Output:
```
[
    {
        "name": "Emma",
        "gender": "Female",
        "year of popularity": 2020,
        "percentage of population": 1.012
    },
    {
        "name": "Liam",
        "gender": "Male",
        "year of popularity": 2020,
        "percentage of population": 1.005
    },
    {
        "name": "Olivia",
        "gender": "Female",
        "year of popularity": 2020,
        "percentage of population": 0.889
    }
]
```

**Tactic 3:**
- Ask the model to check whether conditions are satisfied.\
Example:
```python
   prompt = f"""
   You will be provided text delimited by <>. If it contains the diagnoses, symptoms, and medications, generate a JSON with the following keys:
   diagnoses, symptoms, and medications.
   If only partial information exists, then write "Incomplete information". If there is no information, write "No information".

   <Patient is a 34-year-old female presenting with intermittent chest tightness and shortness of breath over the last month,
   particularly during periods of stress or exertion. No syncope, no history of cardiac disease. EKG today normal, lungs clear,
   O2 sat 98% on room air. Likely anxiety-related symptoms, discussed non-pharmacologic interventions including breathing exercises
   and mindfulness. Patient hesitant about medication but agreed to trial of propranolol 10 mg PRN for symptomatic relief.
   Will refer to behavioral health. Follow-up scheduled in 2 weeks. No current suicidal ideation or red flags noted.>
   """
```
Output:
```
{
  "diagnoses": "Likely anxiety-related symptoms",
  "symptoms": "Intermittent chest tightness and shortness of breath",
  "medications": "Propranolol 10 mg PRN"
}
```

**Tactic 4: "Few-shot" prompting**
- Give successful examples of completing tasks before asking the model to perform the task.\
Example:
```python
   prompt = f"""
   Your task is to answer in a consistent style.
   <child>: Teach me about patience.
   <grandparent>: The river that carves the deepest valley flows from a modest spring;
   the grandest symphony originates from a single note;
   the most intricate tapestry begins with a solitary thread.

   <child>: Teach me about resilience.
   """
```
Output:
```
<grandparent>: Resilience is like a mighty oak tree that withstands the fiercest storms, bending but never breaking.
It is the inner strength that allows us to bounce back from adversity, like a phoenix rising from the ashes.
```

### Principle 2: Give the model time to "think"
**Tactic 1:**
- Specify the steps required to complete a task. Ask for output in a specified format if necessary.\
Example:
```python
   text = f"""
   Juno the frog dreamed of adventure, and his best friend Jennifer the bird was ready to fly. Together, they set out
   to find the legendary Rainbow Gem atop Whistlewind Mountain. They crossed squishy valleys, tiptoed past a
   sleeping bear, and solved a wise owl’s riddle. At the mountain’s peak, they found the glowing gem. Instead of keeping it,
   they brought it back to Lilyville, where its light made the forest bloom forever. Juno and Jennifer
   became heroes—not just for their bravery, but for their kindness. And from that day on, adventure was always just a hop or flap away.
   """

   prompt = f"""
   Perform the following actions:
   1. Summarize the following story delimited by <> in less than 20 words.
   2. List each character's name.
   3. Translate the summary into French.
   4. Create a JSON object that contains the following keys: story, num_characters, and french_summary
   5. Separate your answers with line breaks.
   
   Output only the JSON object.
   
   story = <{text}>
   """
```
Output:
```
{
  "story": "Juno and Jennifer seek Rainbow Gem, become heroes with kindness.",
  "num_characters": 2,
  "french_summary": "Juno et Jennifer cherchent Rainbow Gem, deviennent héros avec gentillesse."
}
```

**Tactic 2:**
- Instruct the model to work out its own solution before rushing to a conclusion.\
Example:
```python
prompt = f"""
   Your task is to determine whether a student's solution below delimited by <> to a math problem below delimited by <>
   is correct or not.
   Before making the determination, you will make your own calculations and come up with a solution.
   Then you will compare your solution with the student's and make the determination.
   Write the output as follows:

    Math problem: math problem
  
    Student's solution: student's solution
  
    Solution: Your calculated solution
  
    Comparison: Do the answers match? Output "Yes" if they match, or "No" if they don't match.
  
    Student's grade: Correct or Incorrect.
  
    Math question:
    <A bakery sells cookies in boxes of 6. If Lily buys 4 boxes, how many cookies does she have in total?>
  
    Student's Solution:
    <Lily buys 4 boxes and each box has 6 cookies, so: 4 + 6 = 10 cookies.>
    """
```
Output:
```
Math problem: A bakery sells cookies in boxes of 6. If Lily buys 4 boxes, how many cookies does she have in total?

Student's solution: Lily buys 4 boxes and each box has 6 cookies, so: 4 + 6 = 10 cookies.

Solution: 4 boxes * 6 cookies per box = 24 cookies

Comparison: No

Student's grade: Incorrect
```

## Model Limitations:
- Hallucinations: Models make statements that sound plausible but are not true.
Example:
```python
   prompt = f"""
   Write a 50 word review of the Juniper Bloom perfume by Hermes
   """
```
The output is
```
Juniper Bloom by Hermes is a fresh and invigorating scent that captures the essence of a blooming juniper tree.
The fragrance is light and airy, perfect for everyday wear. It has a subtle sweetness with hints of citrus and floral notes.
Overall, a delightful and uplifting perfume for any occasion.
```
- The model generates a very believable review of a fictional product - Juniper Bloom perfume, from a real company - Hermes.

## Iterative prompt development
- The first prompt will hardly ever be the prompt that goes into production
- The prompt can iteratively be analyzed and refined as needed
- Here, I will iteratively refine prompts to generate a marketing product description from a product fact sheet.

```
fact_sheet_desk = """
OVERVIEW
Part of a sophisticated family of mid-century inspired office furniture, including desks, credenzas, bookcases, and file cabinets.
Crafted from sustainably sourced black walnut wood with a rich, hand-applied oil finish.
Available in multiple configurations including single pedestal, double pedestal, and minimalist writing desk.
Sleek silhouette with beveled edges and concealed cable management.
Suitable for home office, executive suites, or collaborative workspaces.
Designed to complement both classic and modern interiors.
Qualified for contract and commercial use.

CONSTRUCTION
Constructed from solid black walnut and veneered engineered hardwood panels.
Mortise-and-tenon joinery ensures long-term durability.
Integrated cable management channel with rear cutout for discreet cord routing.
Optional drawer locks and under-mount soft-close slides for smooth functionality.

DIMENSIONS (Standard Single Pedestal)
WIDTH 140 CM | 55.12”
DEPTH 70 CM | 27.56”
HEIGHT 75 CM | 29.53”
LEG CLEARANCE 66 CM | 25.98”

OPTIONS
Available in three configurations: writing desk, single pedestal, double pedestal.
Optional built-in wireless charging pad.
Drawers available with wood or brushed steel handles.
Custom sizing available upon request.

MATERIALS
TOP & FRAME
Solid black walnut wood with a natural matte oil finish.
Veneered MDF core for strength and warping resistance in large surfaces.

HARDWARE
Brushed steel or matte black metal hardware.
Full-extension, soft-close drawer glides.

CABLE MANAGEMENT
Rear cutout and hidden tray for organized cord routing.

COUNTRY OF ORIGIN
Italy
"""
prompt = f"""
Generate a product description for the following fact sheet delimited by <> for a \
marketing team to use for a product listing on an e-commerce website.

product_description = <{fact_sheet_desk}
"""
```
Output:
```
Introducing our sophisticated and versatile mid-century inspired office furniture collection, crafted from sustainably
sourced black walnut wood with a rich hand-applied oil finish. The sleek silhouette and beveled edges of our desks,
credenzas, bookcases, and file cabinets are designed to complement both classic and modern interiors, making them
suitable for home offices, executive suites, or collaborative workspaces.

Constructed with solid black walnut and veneered engineered hardwood panels, our furniture features mortise-and-tenon
joinery for long-term durability. The integrated cable management channel with rear cutout ensures discreet cord routing,
while optional drawer locks and under-mount soft-close slides provide smooth functionality.

Available in multiple configurations including single pedestal, double pedestal, and minimalist writing desk, our furniture
can be customized with optional built-in wireless charging pads and a choice of wood or brushed steel handles for the drawers.
Custom sizing is also available upon request.

Made in Italy, our furniture is not only stylish and functional but also qualified for contract and commercial use.
Upgrade your workspace with our elegant and practical office furniture collection today.
```
#### Issue 1: The text is too long
- Limit the number of words/characters/sentences
```
prompt = f"""
Generate a product description for the following fact sheet delimited by <> for a \
marketing team to use for a product listing on an e-commerce website.

Limit it to a maximum of 50 words.

product_description = <{fact_sheet_desk}
"""
```
Output:
```
Elevate your workspace with our sophisticated mid-century inspired office furniture collection. Crafted from
sustainably sourced black walnut wood with sleek beveled edges and concealed cable management. Available in multiple
configurations and suitable for home offices, executive suites, or collaborative workspaces. Designed to complement
classic and modern interiors. Made in Italy.
```

#### Issue 2: Text focuses on the wrong details
- Ask the model to focus on the aspects relevant to the intended audience.
```
prompt = f"""
Generate a product description for the following fact sheet delimited by <> for a \
marketing team to use for a product listing on an e-commerce website.

The intended audience is furniture retailers, so focus on the technical aspects such as material, construction, etc.

Limit it to a maximum of 50 words.

product_description = <{fact_sheet_desk}
"""
```
Output:
```
Introducing our sophisticated mid-century inspired office furniture collection, crafted from sustainably sourced black walnut wood with a hand-applied oil finish. Featuring sleek design, durable construction, and integrated cable management, this versatile furniture is perfect for home offices, executive suites, or collaborative workspaces. Made in Italy, this collection is designed to complement both classic and modern interiors.
```

#### Issue 3: The description needs to include the dimensions
```
prompt = f"""
Generate a product description for the following fact sheet delimited by <> for a \
marketing team to use for a product listing on an e-commerce website.

The intended audience is furniture retailers, so focus on the technical aspects such as material, construction, etc.

Limit the description to a maximum of 50 words.

At the end of the description, print out a table of dimensions and name it "Product Dimensions"

Format everything in HTML that can be used in a website.

product_description = <{fact_sheet_desk}
"""
```
Output:
```
<!DOCTYPE html>
<html>
<head>
<title>Product Description</title>
</head>
<body>

<h2>Product Description</h2>
<p>Part of a sophisticated family of mid-century inspired office furniture, crafted from sustainably sourced black walnut wood with a rich, hand-applied oil finish. Available in multiple configurations with concealed cable management.</p>

<h2>Product Dimensions</h2>
<table>
  <tr>
    <th>WIDTH</th>
    <th>DEPTH</th>
    <th>HEIGHT</th>
    <th>LEG CLEARANCE</th>
  </tr>
  <tr>
    <td>140 CM | 55.12”</td>
    <td>70 CM | 27.56”</td>
    <td>75 CM | 29.53”</td>
    <td>66 CM | 25.98”</td>
  </tr>
</table>

</body>
</html>
```

## Summarizing
- AI can be used to extract information from large amounts of text and summarize it into a digestible chunk of information.
- Here, I will interatively summarize text to focus on specific that are relevant

- Text to summarize:
```
prod_review = """
The laser measure arrived two days early, well-packaged and in perfect condition. It’s impressively accurate and made
measuring my living room a quick, one-person job. The display is clear and responsive, though the laser is tough to see
in direct sunlight and the manual could be more user-friendly. The included case feels cheap and lacks a belt clip,
but the device itself is solid and works great. Overall, a great value for the price, especially with fast shipping and
reliable performance.
"""
```

- Summarize with a word/sentence/character limit
```
prompt = f"""
Your task is to generate a short summary of a product review on an ecommerce site.

Summarize the review below delimited by <> in about 30 words maximum.

Review = <{prod_review}>
"""
```
- Output:
```
Impressively accurate laser measure with clear display, though tough to see in sunlight.
Solid device, fast shipping, and reliable performance make it a great value.
```

- Summarize with a focus on shipping and delivery
```
prompt = f"""
Your task is to generate a short summary of a product review on an ecommerce site.

Summarize the review below delimited by <> in about 30 words maximum. Only focus on and include
aspects that mention shipping and delivery of the product.

Review = <{prod_review}>
"""
```
- Output:
```
Fast shipping, well-packaged, accurate laser measure. Clear display, tough to see laser in sunlight.
Cheap case, solid device. Overall great value with reliable performance.
```

- Summarize with a focus on price and value
```
prompt = f"""
Your task is to generate a short summary of a product review on an ecommerce site.

Summarize the review below delimited by <> in about 30 words maximum. Only focus on and include
aspects that mention price and value of the product.

Review = <{prod_review}>
"""
```
- Output:
```
Great value for the price, accurate and easy to use. Display is clear, but laser is hard to see
in sunlight. Cheap case, but solid device.
```

**Notes: As the output indicates, summarizing includes topics that are not related to the topic of focus.**

## **Using "extract" instead of "summarize"**
```
prompt = f"""
Your task is to extract relevant information from a product review on an e-commerce website to give feedback
to the shipping department. 
From the review below delimited by <>, extract information with keywords shipping and delivery only.
Limit it to 10 words.

Review: <{prod_review}>
"""
```
Output:
```
Feedback: 
Fast shipping, arrived two days early, well-packaged, reliable performance.
```

### Summarizing multiple product reviews
```
review_1 = prod_review

# Review for a candy thermometer
review_2 = """
This thermometer arrived quickly and was well-protected in the packaging. It clipped easily onto my saucepan and gave quick,
consistent readings while I made toffee and fudge. The markings are clear, but I wish the numbers were a bit larger for easier
reading through steam. It feels durable and washes up easily, though it doesn’t come with a case for storage. Still, it’s a
reliable kitchen tool that made my candy-making stress-free.
"""

# Review for a hand cream
review_3 = """
Shipping was fast, and the jar arrived sealed and in perfect condition. This cream is super hydrating without feeling greasy,
and my skin stays soft for hours after applying. The scent is light and fresh—not overpowering like some lotions I’ve tried.
I’ve noticed a real improvement in dry patches, especially on my elbows and knees. It’s become a staple in my nighttime routine
and I’ll definitely repurchase.
"""

# Review for a foot massager
review_4 = """
The massager arrived on time and was ready to use straight out of the box. It offers strong, soothing pressure and the heat
feature makes a big difference after a long day. It’s a bit bulky to store, and it can be noisy at higher settings, but it works
incredibly well. My feet feel lighter and more relaxed after just 15 minutes. If you’re on your feet all day, this is a game-changer.
"""

reviews = [review_1, review_2, review_3, review_4]
```

To summarize all four reviews at the same time:
```
for i in range(len(reviews)):
   prompt = f"""
   Your task is to generate a short summary of a product review on an ecommerce site.

   Summarize the review below delimited by <> in about 15 words maximum.

   Review = <{reviews[i]}>
   """
```
Output:
```
0 <Accurate, fast, and reliable laser measure with clear display, minor issues with visibility and manual.> 

1 <Quick, consistent thermometer for candy-making, clear markings but wish numbers were larger for easier reading. Durable.> 

2 <Fast shipping, hydrating cream with light scent, improves dry patches, staple in nighttime routine. Will repurchase.> 

3 <Effective massager with strong pressure, soothing heat, and quick relief for tired feet. Game-changer.>
```

## Inferring
- Inferring sentiment

```
foot_massager_review = """
The massager arrived on time and was ready to use straight out of the box. It offers strong,
soothing pressure and the heat feature makes a big difference after a long day. It’s a bit bulky
to store, and it can be noisy at higher settings, but it works incredibly well. My feet feel
lighter and more relaxed after just 15 minutes. If you’re on your feet all day, this is a game-changer.

"""
```
```
prompt = f"""
What is the sentiment of the product review below delimited by <>?
Product review = <{foot_massager_review}>
"""
```
- Output:
```
Positive Sentiment
```

- Inferring emotions:
```
prompt = f"""
Identify five emotions that the writer of the following review, which is delimited by <>, is expressing.
Product review = <{foot_massager_review}>
"""

response = get_completion(prompt)
print(response)
```
- Output:
```
1. Satisfaction
2. Appreciation
3. Frustration
4. Excitement
5. Relief
```

- Identifying anger:
```
prompt = f"""
Is the writer of the following review expressing anger?
The review is delimited by <>.
Answer either yes or no.
Product review = <{foot_massager_review}>
"""

response = get_completion(prompt)
print(response)
```
- Output:
```
No
```

- Extracting product and company name from reviews
```
prompt = f"""
Identify the following items from the review text: 
- Item purchased by reviewer
- Company that made the item

The review is delimited with <>. Format your response as a JSON object with
"Item" and "Brand" as the keys. If the information isn't present, use "unknown"
as the value.
Make your response as short as possible.
  
Review text: <{foot_massager_review}>
"""
```
- Output:
```
{
  "Item": "massager",
  "Brand": "unknown"
}
```

- Doing multiple tasks at once
```
Identify the following items from the product review delimited by <>
- The sentiment of the reviewer - Positive or Negative
- If the reviewer is expressing anger
- Item purchased by reviewer
- Company that made the item

Format your response as a JSON object with "Sentiment", "Angry", 
"Item" and "Brand" as the keys. If the information isn't present, use "unknown"
as the value.
Make your response as short as possible.
Format "Angry" value as a Boolean
Review text: <{foot_massager_review}>
"""
```
- Output:
```
{
    "Sentiment": "Positive",
    "Angry": false,
    "Item": "massager",
    "Brand": "unknown"
}
```

- Inferring topics
```
prompt = f"""
Determine 5 topics that are being discussed in the news article below delimited by <>.
Limit each topic to one or two words.
List the topics seperated by commas.

News Article = <{story}>
"""
```
- Output:
```
Zoo, Penguin, Conservation, Animal Rights, Captivity
```

## Transforming
- LLMs are good at transforming text, for example, translating from one language to another, changing the tone, spellchecking and proofreading, and even converting formats such as switching to JSON or HTML

- Translation
```
prompt = f"""
Translate the following text delimited by <> into French, Spanish, and English pirate.

Text = <I think cats are extrememly cute!>
"""
```
- Ouput:
```
French: <Je pense que les chats sont extrêmement mignons!>
Spanish: <¡Creo que los gatos son extremadamente lindos!>
Pirate: <I reckon cats be right cute!>
```

- Changing tone:
```
prompt = f"""
Change the tone of the following text delimited by <> to polite and formal.

Text = < I don't care that your blender is broken. You can replace it yourself.>
"""
```
- Output:
```
I apologize for the inconvenience of your broken blender. It would be best for you to replace it yourself.
```

- Spellcheck and proofreading
```
prompt = f"""
Proofread the following text delimited by <> and make sure the grammar is correct.

<I don't think I and John will be making it to the diner party on Sunray. We hve so mch chores and things to do.>
"""
```
- Output:
```
I don't think John and I will be making it to the dinner party on Sunday. We have so
many chores and things to do.
```

- Conversion
```
data_json = { "grade 4 students" :[ 
    {"name":"Jill Smith", "grade":"A"},
    {"name":"Jenna Darko", "grade":"B"},
    {"name":"Jai Patel", "grade":"B"}
]}

prompt = f"""
Translate the following python dicionary delimited by <> from JSON to an HTML table with
column headers and title
JSON: <{data_json}>
"""
```
- Output:
```
<!DOCTYPE html>
<html>
<head>
    <title>Grade 4 Students</title>
</head>
<body>

<table border="1">
    <tr>
        <th>Name</th>
        <th>Grade</th>
    </tr>
    <tr>
        <td>Jill Smith</td>
        <td>A</td>
    </tr>
    <tr>
        <td>Jenna Darko</td>
        <td>B</td>
    </tr>
    <tr>
        <td>Jai Patel</td>
        <td>B</td>
    </tr>
</table>

</body>
</html>
```

## Expanding




