## OpenAI - ChatGPT Prompt Engineering for Developers

# Two types of LLMs
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

# Guidelines for prompting:
- Principle 1: Write clear and specific instructions. Clear does not equal short.
- Principle 2: Give the model time to think.

# Principle 1: Write clear and specific instructions
Tactic 1: 
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
   <text>
   """
```
Tactic 2:
- Ask for a structured output like JSON, HTML, etc.\
Example:
```python
   prompt = f"""
   Generate a list of six baby names along with the gender, year of popularity, and how many percentage of the population has that name.
   Provide them in JSON format with the following keys: name, gender, year of popularity, percentage of population.
   """
```
Tactic 3:
- Ask the model to check whether conditions are satisfied.\
Example:
```python
   prompt = f"""
   You will be provided text delimited by <>. If it contains the diagnoses, symptoms, and medications, generate a JSON with the following keys:
   diagnoses, symptoms, and medications.
   If only partial information exists, then write "Incomplete information". If there is no information, write "No information".
   <text>
   """
```
Tactic 4: "Few-shot" prompting
- Give successful examples of completing tasks before asking the model to perform the task.\
Example:
```python
   prompt = f"""
   Your task is to answer in a consistent style.
   <child>: Teach me about patience.
   <grandparent>: The river that carves the deepest valley flows from a modest spring; the grandest symphony originates from a single note;
   the most intricate tapestry begins with a solitary thread.

   <child>: Teach me about resilience.
   """
   response = get_completion(prompt)
   print(response)
```
# Principle 2: Give the model time to "think"
Tactic 1: Specify the steps required to complete a task. Ask for output in a specified format if necessary.\
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
   1. Summarize the following story delimited by <> in less than 30 words.
   2. Translate the summary into French.
   3. List each name in the French summary.
   4. Output a JSON object that contains the following keys: french_summary and num_names
   5. Separate your answers with line breaks.

   story = <{text}>
   """
```
Tactic 2: Instruct the model to work out its own solution before rushing to a conclusion.\
Example:
```python
   math_problem = f"""
   A bakery sells cookies in boxes of 6. If Lily buys 4 boxes, how many cookies does she have in total?"
   """
   student_solution = f"""
   Lily buys 4 boxes and each box has 6 cookies, so:
   4 + 6 = 10 cookies.
   """
   prompt = f"""
   Your task is to determine whether a student's solution to a math problem is correct or not.
   Before making the determination, you will make your own calculations and come up with a solution.
   Then you will compare your solution with the student's and make the determination.
   Write the ouput as follows:
   Math problem: <{math_problem}>
   Student's solution: <{student_solution}>
   Solution: Your calculated solution
   Comparision: Whether the solutions match, "Yes" or "No"
   Student's grade: Correct or Incorrect.
   """
```
# Model Limitations:
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
- The model generates a very believable review of a fictional product - Juniper Bloom perfume from a real company - Hermes.


