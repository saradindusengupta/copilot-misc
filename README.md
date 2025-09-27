# Copilot Miscellaneous

> A comprehensive collection of GitHub Copilot customizations including chat modes, instructions, and prompts for enhanced AI-assisted development workflows.

⭐ If you like this project, star it on GitHub — it helps a lot!

[Features](#features) • [Getting Started](#getting-started) • [Chat Modes](#chat-modes) • [Instructions](#instructions) • [Prompts](#prompts) • [Usage](#usage)

## Overview

This repository contains a curated collection of customizations for GitHub Copilot, designed to enhance your development experience through specialized chat modes, detailed instructions, and powerful prompts. Whether you're debugging code, planning architecture, or generating documentation, these tools help you work more efficiently with AI assistance.

## Features

- **🤖 Specialized Chat Modes**: 12 distinct chat modes tailored for different development scenarios
- **📋 Development Instructions**: Comprehensive guidelines for various technologies and practices
- **🎯 Purpose-Built Prompts**: Ready-to-use prompts for common development tasks
- **🔧 Extensible Framework**: Easy to customize and extend for your specific needs
- **📚 Best Practice Integration**: Built-in adherence to DevOps, performance, and security best practices

## Getting Started

1. **Clone the repository**:
   ```bash
   git clone https://github.com/saradindu-lytx/copilot-misc.git
   cd copilot-misc
   ```

2. **Configure VS Code**: Place the files in your VS Code user data directory to enable the customizations globally, or keep them in your workspace for local use.

3. **Start using**: The chat modes, instructions, and prompts will be available in your GitHub Copilot interface.

> [!TIP]
> For best results, familiarize yourself with the available chat modes and their specific purposes before starting your development tasks.

## Chat Modes

The `chatmodes/` directory contains specialized AI personas for different development scenarios:

| Chat Mode | Description | Best For |
|-----------|-------------|----------|
| **debug** | Debug applications to find and fix bugs | Troubleshooting, error analysis |
| **demonstrate-understanding** | Validate understanding through guided questioning | Code reviews, learning |
| **gilfoyle** | Sardonic technical supremacy code reviewer | Code quality, technical criticism |
| **implementation-plan** | Generate executable implementation plans | Project planning, task breakdown |
| **mentor** | Educational guidance and mentorship | Learning, skill development |
| **microsoft-learn-contributor** | Microsoft Learn content creation | Documentation, tutorials |
| **plan** | Strategic planning and architecture assistant | Architecture, system design |
| **principal-software-engineer** | Senior-level technical guidance | Complex problem solving |
| **prompt-builder** | Expert prompt engineering and validation | AI prompt optimization |
| **software-engineer-agent** | Comprehensive development assistance | General development tasks |
| **specification** | Generate technical specifications | Requirements, documentation |
| **task-researcher** | Comprehensive project analysis | Research, investigation |
| **tech-debt-remediation-plan** | Technical debt remediation planning | Code improvement, refactoring |

## Instructions

The `instructions/` directory provides detailed guidelines for various aspects of development:

- **`containerization-docker-best-practices.instructions.md`**: Docker optimization and security
- **`devops-core-principles.instructions.md`**: CALMS framework and DORA metrics
- **`gilfoyle-code-review.instructions.md`**: Sardonic but accurate code review style
- **`github-actions-ci-cd-best-practices.instructions.md`**: CI/CD pipeline best practices
- **`performance-optimization.instructions.md`**: Comprehensive performance tuning guide
- **`python.instructions.md`**: Python coding conventions and guidelines

## Prompts

The `prompts/` directory contains ready-to-use prompts for common development tasks:

- **Architecture & Design**: Blueprint generation, ADR creation, breakdown prompts
- **Documentation**: README creation, folder structure documentation
- **Code Review**: SQL optimization, safety reviews
- **Project Management**: Git flow, epic breakdown, test planning

## Usage

### Using Chat Modes

1. Open GitHub Copilot Chat in VS Code
2. Reference the desired chat mode in your conversation
3. The AI will adopt the specialized persona and follow the mode's guidelines

**Example**:
```
@copilot /mode debug
Help me find the bug in this authentication function
```

### Applying Instructions

Instructions are automatically applied based on file patterns. For example:
- Python files (`.py`) automatically use Python coding conventions
- Docker files use containerization best practices
- GitHub Actions workflows use CI/CD guidelines

### Using Prompts

Copy and customize prompts from the `prompts/` directory for your specific needs:

```bash
# Example: Generate a README for your project
cp prompts/create-readme.prompt.md my-project/
# Customize the prompt and use with Copilot
```

## Key Features by Use Case

### **Code Quality & Review**
- Gilfoyle chat mode for technical criticism
- Performance optimization instructions
- Code review prompts with safety checks

### **Project Planning**
- Implementation plan generation
- Architecture blueprint creation
- Epic and feature breakdown prompts

### **DevOps & Deployment**
- Docker best practices
- GitHub Actions CI/CD guidelines
- Performance monitoring and optimization

### **Documentation**
- Automated README generation
- Technical specification creation
- Architecture decision records

## Advanced Usage

### Customizing Chat Modes

Each chat mode is defined with specific tools, descriptions, and behavioral guidelines. You can:

1. **Modify existing modes**: Edit the markdown files to adjust behavior
2. **Create new modes**: Follow the existing format and add your specialized requirements
3. **Combine modes**: Reference multiple modes for complex scenarios

### Extending Instructions

Instructions use pattern matching to apply automatically:

```yaml
---
applyTo: '**/*.py'  # Applies to all Python files
description: 'Python coding conventions and guidelines'
---
```

### Building Custom Prompts

Follow the prompt template structure:

```yaml
---
mode: 'agent'
description: 'Your prompt description'
---

## Role
[Define the AI's role and expertise]

## Task
[Specific instructions and requirements]
```

## Best Practices

- **Start with Planning**: Use planning modes before implementation
- **Layer Your Approach**: Combine instructions with appropriate chat modes
- **Iterate and Refine**: Use prompt-builder mode to optimize your prompts
- **Document Decisions**: Use specification mode for important architectural choices

> [!NOTE]
> These customizations are designed to work together. The instructions inform the chat modes, and the prompts provide starting points for complex tasks.

## Resources

- [GitHub Copilot Documentation](https://docs.github.com/copilot)
- [VS Code Extensions](https://marketplace.visualstudio.com/vscode)
- [DevOps Best Practices](https://docs.microsoft.com/azure/devops/)
- [Performance Optimization Guides](https://web.dev/performance/)

---

This project is designed to enhance your development workflow through AI assistance. Each component works together to provide a comprehensive, intelligent development environment that adapts to your needs and follows industry best practices.