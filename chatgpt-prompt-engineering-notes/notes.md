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
- 
