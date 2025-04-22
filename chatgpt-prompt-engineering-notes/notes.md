## OpenAI - ChatGPT Prompt Engineering for Developers

# Two types of LLMs
1. Base LLM: Predicts next word based on text training data
   Example:
   a) **Once upon a time, there was a unicorn** _that lived in a magical forest with all her unicorn friends_
   b) **What is the capital of France?**
      _What is France's largest city?_
      _What is France's population?_
      _What is the currency of France?_
   
   - A base LLM would not answer the question but rather complete the text that was given based on its training data. In this case, the data may be a list of exam questions.
     
2. Instruction Tuned LLM - tries to follow instructions
   Example:
   **What is the capital of France?**
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
- Delimiters can be any kind of clear punctuation that seperate specific pieces of text from the rest of the prompt. This makes it clear to the model that this is a separate section.
- Delimiters are also useful in preventing prompt injections.
Example:  text = f"""
          A long text of over 50 words that needs to be summarized.
          """
          prompt = f"""
          Summarize the following text delimited by <> into one sentence with less than 20 words.
          <text>
          """
  
Tactic 2:
- Ask for a structured output like JSON, HTML, etc.
Example:  prompt = f"""
          Generate a list of six baby names along with the gender, year of popularity, and how many percentage of the population has that name. Provide them in JSON format with the following keys: name, gender, year of popularity, percentage of population.
"""

Tactic 3:
- Ask the model to check whether conditions are satisfied.
Example:  prompt = f"""
          You will be provided text delimited by <>. If it contains the diagnoses, symptoms, and medications, generate a JSON with the following keys:
          diagnoses, symptoms, and medications.
          If only partial information exists, then write "Incomplete information". If there is no information, write "No information".
          <text>
          """

Tactic 4: "Few-shot" prompting
- Give successful examples of completing tasks before asking the model to perform the task.
Example:  prompt = f"""
          Your task is to answer in a consistent style.

          <child>: Teach me about patience.

          <grandparent>: The river that carves the deepest valley flows from a modest spring; the grandest symphony originates from a single note; the most intricate tapestry begins with a solitary thread.

          <child>: Teach me about resilience.
          """
          response = get_completion(prompt)
          print(response)

  # Principle 2: Give the model time to "think"
  Tactic 1: 
