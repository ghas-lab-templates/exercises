# Style Guide for Lab Instructions

## General Guidelines
- **Structure**: Use consistent heading levels (`#`, `##`, `###`) to organize content logically.
- **Language**: Use clear, concise, and action-oriented language. Address the user directly with commands such as "click," "navigate," or "select."
- **Formatting**:
  - Use **bold** for emphasis and critical instructions.
  - Use `inline code formatting` for command names, file paths, or UI elements.
  - Use numbered lists for step-by-step instructions.
  - Use bullet points for non-sequential items.
  - Embed images or gifs for visual aid where necessary, enclosed within `<details>` tags for optional viewing.

## Content Sections
### 1. Labs
Each lab should include:
- **Title**: Use `### Lab [number] - [Lab Name]`.
- **Objective**: Briefly describe the goal or purpose of the lab.
- **Steps**: Use numbered lists for step-by-step tasks. For complex tasks, include explanatory text before each step. Use `<details>` tags for optional help or visuals, e.g., screenshots or animated guides.
- **Discussion Points**: Use bullet points for each discussion point
- 
### 2. Code Snippets
- Place code snippets in triple backticks (\`\`\`) with the appropriate language identifier for syntax highlighting.
- Include comments within code snippets to explain key parts.
- For extended snippets or files, consider wrapping them in `<details>` tags with a summary header like `Solution: workflow file`.
- Use inline code formatting for single commands or file paths.
- Use fenced code blocks for multi-line commands. For example:
```bash
git add .
git commit -m "Initial commit"
git push
```

### 3. Discussion Points
- Include a discussion section at the end of relevant labs to encourage critical thinking and application.
- Use bullet points for questions or topics, starting with action verbs like "Consider," "Discuss," or "Evaluate."

### 4. Visual Aids
- Embed images or gifs using Markdown syntax:
- Enclose visual aids within `<details>` tags for optional viewing:
```markdown
<details>
  <summary>Animated Guide</summary>
  ![alt text](path/to/image.gif)
</details>
```
### 5. Example Lab Layout

### Lab X - [Lab Name]

#### Objective
[Brief description of the goal of the lab.]

#### Steps
1. [Actionable instruction.]
   - [Additional clarification if necessary.]
2. [Next step.]
   <details>
     <summary>Need Help? View Screenshot</summary>
     ![alt text](path/to/image.png)
   </details>

#### Discussion Points
- [Question 1]
- [Question 2]
- [Question 3]


