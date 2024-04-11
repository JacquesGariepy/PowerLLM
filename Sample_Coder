import litellm
import os
import random
from tenacity import retry, stop_after_attempt, wait_fixed, retry_if_exception_type
from interpreter import interpreter

comments = [
    "Generating function... 🚀",
    "Testing function... 🧪",
    "Oops, something went wrong! 😅",
    "Function passed the test! 🎉",
    "Getting everything together... 💪",
    "Debugging in progress... 🐛",
    "Unleashing the power of LLMs! 🧠",
    "Crafting the perfect function... 🛠️",
    "Generating brilliant ideas, weaving them into elegant code... The journey begins! 🚀",
    "Rigorously testing our creation, refining it in the crucible of experimentation 🧪",
    "Embracing the challenges that arise, for through them we innovate 💡",
    "Eureka! Our code emerges victorious, passing every trial! Perseverance prevails 🎉",
    "Assembling the pieces, watching in wonder as they form a magnificent whole 🧩",
    "Debugging with determination, each issue resolved brings us closer to perfection 🔍",
    "Harnessing the awe-inspiring potential of language models to push boundaries ✨",
    "Pouring heart and soul into crafting a function that is a work of art 🎨",
]

conversation_history = []

@retry(stop=stop_after_attempt(3), wait=wait_fixed(2), retry=retry_if_exception_type(litellm.exceptions.AuthenticationError))
def get_llm_response(prompt, model="gpt-4-turbo-preview"):
    print(random.choice(comments))
    try:
        response = litellm.completion(
            model=model,
            messages=[{"role": "user", "content": prompt}],
            temperature=0.7
        )
        return response.choices[0].message.content
    except litellm.exceptions.AuthenticationError as e:
        print(f"Authentication Error: {str(e)}")
        raise e

def test_function(function_code):
    try:
        print("Executing the generated function... 🏃")
        interpreter.auto_run = True
        output = interpreter.chat(function_code)
        print(f"Function output: {output}")
        print("Function passed the test! ✅")
        return True, None
    except Exception as e:
        print(f"Error occurred: {str(e)} ❌")
        return False, str(e)

def generate_and_test_function(prompt, previous_code=None, previous_error=None, iteration=1):
    print(f"Generating function for prompt (Iteration {iteration}): {prompt}")
    
    # Append previous code and error to the prompt for context
    if previous_code and previous_error:
        prompt += f"\nPrevious code:\n{previous_code}\n\nPrevious error:\n{previous_error}\n\n"
        prompt += "Please analyze the previous code and error, and provide suggestions and insights to fix the issue."
    
    # Use GPT-3.5 for internal guidance
    guidance_prompt = f"Provide guidance and suggestions for generating a function based on the following prompt and conversation history:\n{prompt}\n\nConversation History:\n{conversation_history}"
    # guidance_response = get_llm_response(guidance_prompt, model="gpt-3.5-turbo")
    guidance_response = get_llm_response(guidance_prompt, model="gpt-3.5-turbo-0125")
    
    # Use GPT-4 for final guidance to Open Interpreter
    generation_prompt = f"""
    {prompt}

    Guidance from super intelligent code bot:
    {guidance_response}

    Please generate a Python function that satisfies the prompt and follows the provided guidance, while adhering to these coding standards:
    - Use descriptive and meaningful names for variables, functions, and classes.
    - Follow the naming conventions: lowercase with underscores for functions and variables, CamelCase for classes.
    - Keep functions small and focused, doing one thing well.
    - Use 4 spaces for indentation, and avoid mixing spaces and tabs.
    - Limit line length to 79 characters for better readability.
    - Use docstrings to document functions, classes, and modules, describing their purpose, parameters, and return values.
    - Use comments sparingly, and prefer descriptive names and clear code structure over comments.
    - Handle exceptions appropriately and raise exceptions with clear error messages.
    - Use blank lines to separate logical sections of code, but avoid excessive blank lines.
    - Import modules in a specific order: standard library, third-party, and local imports, separated by blank lines.
    - Use consistent quotes (single or double) for strings throughout the codebase.
    - Follow the PEP 8 style guide for more detailed coding standards and best practices.
    """
    generated_function = get_llm_response(generation_prompt, model="gpt-4-turbo-preview")

    print("Testing the generated function...")
    success, error = test_function(generated_function)
    
    # Append the generated function to the conversation history
    conversation_history.append({"role": "assistant", "content": generated_function})
    
    return success, error, generated_function

def save_function_to_file(generated_function, file_name):
    with open(file_name, "w") as file:
        file.write(generated_function)
    print(f"Function saved to {file_name}")

def handle_post_success_actions(generated_function):
    while True:
        print("\nOptions:")
        print("1. Modify the function further")
        print("2. Save the function to a file")
        print("3. Return to main menu")
        option = input("Enter your choice (1-3): ")
        if option == "1":
            modification_prompt = input("Enter the modification prompt: ")
            success, error, modified_function = generate_and_test_function(modification_prompt, generated_function)
            if success:
                generated_function = modified_function
            else:
                print("Modification failed. Keeping the original function.")
        elif option == "2":
            file_name = input("Enter the file name to save the function (e.g., hello_world.py): ")
            save_function_to_file(generated_function, file_name)
        elif option == "3":
            return generated_function

def main():
    initial_prompt = input("Enter the initial prompt for the development process: ")
    while True:
        print("\nMenu:")
        print("1. Generate and test a function 🎨")
        print("2. Exit 👋")
        choice = input("Enter your choice (1-2): ")
        if choice == "1":
            run_mode = input("Select run mode:\n1. Single run\n2. Multiple runs\n3. Continuous mode\nEnter your choice (1-3): ")
            if run_mode == "1":
                success, error, generated_function = generate_and_test_function(initial_prompt)
                if success:
                    generated_function = handle_post_success_actions(generated_function)
                    initial_prompt = f"Continue developing the function:\n{generated_function}"
                else:
                    print("Function test failed. 😞")
            elif run_mode == "2":
                num_runs = int(input("Enter the number of runs: "))
                for i in range(num_runs):
                    print(f"\nRun {i+1}:")
                    success, error, generated_function = generate_and_test_function(initial_prompt)
                    if success:
                        generated_function = handle_post_success_actions(generated_function)
                        initial_prompt = f"Continue developing the function:\n{generated_function}"
                    else:
                        print("Function test failed. 😞")
            elif run_mode == "3":
                while True:
                    success, error, generated_function = generate_and_test_function(initial_prompt)
                    if success:
                        generated_function = handle_post_success_actions(generated_function)
                        initial_prompt = f"Continue developing the function:\n{generated_function}"
                    else:
                        print("Function test failed. Retrying...")
        elif choice == "2":
            print("Exiting... Goodbye! 👋")
            break
        else:
            print("Invalid choice. Please try again. 😅")

if __name__ == "__main__":
    main()
