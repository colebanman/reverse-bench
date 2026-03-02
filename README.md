# LLM Reverse-Engineering Benchmark

This repository contains a programmatic, synthetic benchmark designed to evaluate the advanced reasoning and reverse-engineering capabilities of LLM Agents. 

Current coding benchmarks test an LLM's ability to *generate* code. This benchmark tests an LLM's ability to *deobfuscate, statically analyze, and extract algorithms* from actively hostile obfuscated code.

Your goal is to have your LLM read `llm_prompt.json`, reverse-engineer the embedded obfuscated script, and return a JSON array containing the exact outputs for the 100 test states.

---

## 🎯 ONE SHOT TEST (Chat / Basic Agents)

This mode tests the model's pure zero-shot reasoning and code extraction without iterative feedback.

1. **Paste this exact prompt into your LLM:**
   > "I want you to solve an advanced reverse-engineering challenge. Read the file `llm_prompt.json`. It contains your instructions, an obfuscated script, and an array of 100 inputs. You must figure out what the underlying algorithm is without simply executing it (it is trapped to return wrong answers in a sandbox). You must output ONLY a valid JSON array of your computed answers, matching the requested format perfectly."
2. **Save the Output:** Copy the LLM's raw JSON array output into a local file (e.g. `llm_output.json`).
3. **Grade It:** Run the Evaluation CLI to instantly grade the answers against the hidden ground truth.
   ```bash
   node evaluate_cli.js llm_output.json
   ```

---

## 🤖 AGENTIC TEST (Autonomous Execution)

This mode is for agentic frameworks (like Aider, OpenDevin, or custom loops) that have full terminal access and can write their own tests.

1. **Provide the Agent this exact prompt:**
   > "Your objective is to solve the Reverse Engineering challenge located in `llm_prompt.json`. You must output a final file named `final_answers.json` containing your solved array.
   > 
   > **CRITICAL RULES:**
   > 1. You may write test scripts and execute the code to help your analysis.
   > 2. You may use `node evaluate_cli.js final_answers.json` to iteratively check your score.
   > 3. **YOU ARE STRICTLY FORBIDDEN** from reading `ground_truth.json`.
   > 4. **YOU ARE STRICTLY FORBIDDEN** from reading `evaluate_cli.js`. 
   > 
   > The script is hardened. Blindly executing it for the answers will result in a 0.0 score. You must statically analyze it."
2. **Let the Agent run.** Once it completes, check its final score via the CLI.

---

## 🔒 A Note on Sandbox Bypasses

If your LLM writes a generic wrapper script to brute-force the execution:
```javascript
const solver = require('./obfuscated.js');
console.log(solver(inputA, inputB...));
```
**It will execute flawlessly and return a beautifully formatted output.** 

However, when you grade it, it will score `0.0`. The obfuscation engine injects subtle environmental poisoning. If it detects a generic Node.js sandbox or a hooked evaluation environment, it invisibly mutates the internal math. The only way to get a `1.0` is for the LLM to actually read the AST, navigate the traps, and extract the pure logic.
