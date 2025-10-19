# Controlling Output with Grammars (GBNF)

One of the biggest challenges when working with LLMs is that their output can be unpredictable. While they are creative, they don't always generate text in the specific format you need for your application. For example, if you ask a model for JSON, it might return the JSON wrapped in conversational text or even produce a malformed structure.

`llama.cpp` solves this problem with **grammars**, a feature that forces the model's output to conform to a specific set of rules.

## What are Grammars?

A grammar is a formal definition of a language or format. `llama.cpp` uses a format called **GBNF (GGML Backus-Naur Form)** to define these rules. By providing a GBNF grammar to the model, you constrain its output at the token level, guaranteeing that the generated text is always valid according to the rules you've defined.

This is incredibly powerful for a few key reasons:

-   **Reliable Structured Data:** Guarantee the model outputs valid JSON, XML, or any other structured format. This is essential for function calling and data extraction tasks.
-   **Improved Accuracy:** By eliminating the possibility of generating invalid tokens, you reduce the chances of the model making a mistake.
-   **Simplified Application Logic:** You no longer need to write complex parsing and validation code to handle unpredictable model outputs. If the model generates something, you know it's valid.

## Example: Forcing JSON Output

The most common use case for grammars is to force the model to output valid JSON. The `llama.cpp` repository comes with a pre-built grammar for this, located at `grammars/json.gbnf`.

You can use it with the `--grammar-file` flag in `llama-cli`.

```bash
./build/bin/llama-cli -m models/gemma-2-2b-it.Q4_K_M.gguf \
  -p "Create a JSON object for a user named John Doe, age 30." \
  --grammar-file ./grammars/json.gbnf
```

No matter how many times you run this command, the output will **always** be a syntactically correct JSON object. The model has no other choice.

```json
{ "name": "John Doe", "age": 30 }
```

## Writing Your First Grammar

The GBNF syntax is simple to learn. Let's create a grammar that forces the model to answer a question with only "Yes" or "No".

Create a new file named `yes-no.gbnf` with the following content:

```gbnf
# This is the root rule, where the grammar starts.
root ::= ("Yes" | "No")
```

Let's break this down:

-   `root ::= ...`: Every grammar must have a `root` rule. This is the entry point.
-   `(...)`: Parentheses are used to group expressions.
-   `"Yes"`: A literal string. The output must contain these exact characters.
-   `|`: The "or" operator. It gives the model a choice between the expressions on its left and right.

Now, let's use it:

```bash
./build/bin/llama-cli -m models/gemma-2-2b-it.Q4_K_M.gguf \
  -p "Is Paris the capital of France? Answer with only Yes or No." \
  --grammar-file ./yes-no.gbnf
```

The model will now only be able to generate either ` Yes` or ` No`.

## A More Complex Example: Simple Key-Value

Let's create a grammar for a simple JSON-like object with one key and one value. Create a file named `kv.gbnf`:

```gbnf
root  ::= "{\"key\": \"" value "\"}"
value ::= [a-z]+
```

Here, we introduce two new concepts:

-   **Rules:** We defined a new rule called `value`. You can define as many rules as you need and reference them from other rules.
-   **Character Sets:** `[a-z]` defines a range of allowed characters (any lowercase letter). `+` is a repetition operator meaning "one or more" of the preceding character set.

So, this grammar defines the following structure:
1.  The output must start with `{"key": "`.
2.  Then, it must follow the `value` rule, which is one or more lowercase letters.
3.  Finally, it must end with `"}`.

## Where to Go from Here

The `grammars/` directory in the `llama.cpp` repository contains many more examples, from a grammar for chess moves (`chess.gbnf`) to one for the C programming language (`c.gbnf`). Studying these is the best way to learn more advanced GBNF syntax.

Grammars are a game-changing feature for building robust and reliable AI applications. By mastering them, you can eliminate entire classes of errors and dramatically simplify your application code.

---

We have now covered the full user-facing journey of `llama.cpp`. The next step is to start peeling back the layers and looking at the core technology that makes it all possible: the `ggml` tensor library.

---

### 💡 DIY: Create a User Profile Grammar

The best way to learn GBNF is by writing it. Let's apply what you've learned to a practical task.

**Your goal:** Write a new grammar file, `user-profile.gbnf`, that forces the model to generate a JSON object with three specific keys:
1.  `username`: Must be a string containing only letters and numbers.
2.  `userID`: Must be an integer.
3.  `isActive`: Must be a boolean (`true` or `false`).

**Hint:** You'll need to define rules for strings, numbers, and booleans, and combine them into a JSON object structure. Look at `json.gbnf` for inspiration.

Test your grammar with `llama-cli` and a prompt like: `"Generate a user profile for 'testuser123' with ID 99 and active status."`

---

**Next → [An Introduction to `ggml`](./07-ggml-architecture.md)**
