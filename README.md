# Shell Scripting Mastery

**A comprehensive, open-source course for mastering shell scripting on Linux and macOS**

[![Course Status](https://img.shields.io/badge/status-complete-success.svg)](https://github.com/yourusername/shell-scripting-mastery)
[![Units](https://img.shields.io/badge/units-10-blue.svg)](#course-structure)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

## About This Course

This is a complete, production-ready course designed to take you from command-line beginner to advanced shell scripting practitioner. Every unit includes detailed explanations, commented code examples, real-world scenarios, and hands-on exercises.

### What You'll Learn

- **Command Line Mastery** - Navigate and manage files with confidence
- **Shell Scripting Fundamentals** - Variables, functions, and control structures
- **Text Processing** - Master grep, sed, awk, and regex patterns
- **Advanced Techniques** - Arrays, error handling, traps, and signals
- **System Administration** - Package management, cron jobs, and automation
- **Real-World Projects** - Build production-ready scripts and tools
- **Best Practices** - Write secure, maintainable, professional code

### Target Audience

- **Beginners** - No prior experience required
- **Developers** - Automate your workflow
- **System Administrators** - Manage servers efficiently
- **DevOps Engineers** - Build deployment pipelines

### Supported Shells

- **Bash** - Standard on most Linux distributions
- **Zsh** - Default on macOS (Catalina and later)
- Scripts are written for maximum portability

---

## Course Structure

### Core Units

#### [📖 Unit 1: The Command Line Interface (CLI) Fundamentals](course_material/Unit_1_CLI_Fundamentals.md)
**Estimated Time:** 2-3 hours
Learn the difference between terminals and shells, navigate the file system, manage files and directories, and understand Unix permissions.

**Topics Covered:**
- Terminal vs. Shell
- Navigation (`pwd`, `cd`, `ls`)
- File operations (`cp`, `mv`, `rm`, `mkdir`)
- Permissions and `chmod`

---

#### [📖 Unit 2: Introduction to Shell Scripting](course_material/Unit_2_Shell_Scripting_Basics.md)
**Estimated Time:** 2-3 hours
Master the fundamentals of shell scripting including variables, quoting rules, command substitution, and exit codes.

**Topics Covered:**
- The shebang (`#!/bin/bash`)
- Variables and data types
- Quoting rules (single vs. double quotes)
- Command substitution `$()`
- Exit codes and error handling

---

#### [📖 Unit 3: Control Structures and Flow Control](course_material/Unit_3_Control_Flow.md)
**Estimated Time:** 2-3 hours
Make your scripts intelligent with conditional logic and loops.

**Topics Covered:**
- `if/else/elif` statements
- Test conditions (files, strings, numbers)
- `for` loops and iteration
- `while` loops
- Combining control structures

---

#### [📖 Unit 4: Functions and Script Organization](course_material/Unit_4_Functions_and_Organization.md)
**Estimated Time:** 2-3 hours
Write modular, reusable code with functions and proper script organization.

**Topics Covered:**
- Defining and calling functions
- Function arguments
- Variable scope (global vs. local)
- Return values
- The "main" function pattern

---

#### [📖 Unit 5: Input/Output Handling and Text Processing](course_material/Unit_5_IO_and_Text_Processing.md)
**Estimated Time:** 3-4 hours
Master the three standard streams and the "Power Trio" of text processing.

**Topics Covered:**
- STDIN, STDOUT, STDERR
- Redirection operators (`>`, `>>`, `2>`, `&>`)
- Pipes (`|`)
- `grep` - Search text
- `sed` - Stream editor
- `awk` - Pattern processing

---

#### [📖 Unit 6: String Manipulation and Regex](course_material/Unit_6_Strings_and_Regex.md)
**Estimated Time:** 2-3 hours
Manipulate strings and master regular expressions for pattern matching.

**Topics Covered:**
- String slicing and substitution
- Parameter expansion
- Regular expressions
- Pattern matching
- Validation with regex

---

#### [📖 Unit 7: Advanced Scripting Topics](course_material/Unit_7_Advanced_Scripting_Topics.md)
**Estimated Time:** 3-4 hours
Level up with arrays, associative arrays, and robust error handling.

**Topics Covered:**
- Indexed arrays
- Associative arrays (hash maps)
- Error handling with `trap`
- Signal handling (EXIT, INT, TERM)
- Resource cleanup

---

#### [📖 Unit 8: Package Managers and System Administration](course_material/Unit_8_Package_Managers_and_System_Admin.md)
**Estimated Time:** 2-3 hours
Write cross-platform scripts that work with different package managers and automate system tasks.

**Topics Covered:**
- `apt` (Debian/Ubuntu)
- `yum`/`dnf` (RedHat/CentOS)
- `brew` (macOS)
- Cron scheduling
- System monitoring

---

#### [📖 Unit 9: Practical Projects](course_material/Unit_9_Practical_Projects.md)
**Estimated Time:** 4-6 hours
Apply everything you've learned by building real-world automation projects.

**Projects Include:**
- System health monitor
- Automated backup system
- Log analyzer
- Deployment automation
- File organizer

---

#### [📖 Unit 10: Best Practices and Advanced Techniques](course_material/Unit_10_Best_Practices_and_Advanced_Techniques.md)
**Estimated Time:** 3-4 hours
Write production-ready, secure, and maintainable shell scripts.

**Topics Covered:**
- Script structure and templates
- Security best practices
- Debugging techniques
- Performance optimization
- Code review checklist
- ShellCheck and linting

---

### Reference Materials

#### [📚 Reference: Syntax, Commands & Package Management](course_material/Reference_Syntax_Commands_PackageManagement.md)
A comprehensive technical manual covering shell syntax rules, core commands, and package management across different systems.

---

### Complete Curriculum

For a condensed overview of all topics, see the [📋 Full Curriculum Document](curriculum.md).

---

## Getting Started

### Prerequisites

**System Requirements:**
- A Unix-based system (Linux, macOS, or WSL on Windows)
- Terminal emulator (Terminal.app, iTerm2, GNOME Terminal, etc.)
- Text editor (VS Code, vim, nano, etc.)

**Knowledge Requirements:**
- Basic computer literacy
- Willingness to learn
- No prior programming experience required!

### Installation

1. **Clone this repository:**
   ```bash
   git clone https://github.com/yourusername/shell-scripting-mastery.git
   cd shell-scripting-mastery
   ```

2. **Check your shell:**
   ```bash
   echo $SHELL
   ```
   You should see `/bin/bash` or `/bin/zsh`

3. **Start learning!**
   Begin with [Unit 1](course_material/Unit_1_CLI_Fundamentals.md)

### How to Use This Course

1. **Follow in Order** - Units build on each other sequentially
2. **Read Carefully** - Each unit includes detailed explanations
3. **Try the Examples** - Type out the code snippets yourself
4. **Do the Exercises** - Practice is essential for mastery
5. **Build Projects** - Apply concepts in Unit 9's practical projects

### Creating Your First Script

```bash
# Create a new script file
nano hello.sh

# Add this content:
#!/bin/bash
echo "Hello, World!"

# Make it executable
chmod +x hello.sh

# Run it
./hello.sh
```

---

## Course Features

### ✅ Comprehensive Coverage
All fundamental and advanced topics with no gaps

### ✅ Commented Code Examples
Every code snippet includes inline explanations

### ✅ Real-World Scenarios
Learn through practical, production-ready examples

### ✅ Hands-On Exercises
Practice problems at the end of each unit

### ✅ Best Practices
Learn professional coding standards from the start

### ✅ Common Mistakes
See typical errors and how to avoid them

### ✅ Progressive Difficulty
Gentle learning curve from basics to advanced

---

## Learning Path

```
Unit 1 → Unit 2 → Unit 3 → Unit 4 → Unit 5
   ↓
Unit 6 → Unit 7 → Unit 8 → Unit 9 → Unit 10
   ↓
Build Your Own Projects!
```

**Estimated Total Time:** 25-35 hours

---

## Tips for Success

1. **Practice Daily** - Even 30 minutes a day is better than cramming
2. **Type Everything** - Don't copy-paste; muscle memory matters
3. **Break Things** - Experiment and learn from errors
4. **Read Error Messages** - They tell you exactly what's wrong
5. **Use ShellCheck** - A static analysis tool for catching bugs
6. **Build Projects** - Apply your knowledge to real problems
7. **Join Communities** - Ask questions on forums and Discord

---

## Additional Resources

### Recommended Tools

- **[ShellCheck](https://www.shellcheck.net/)** - Catch bugs and bad practices
- **[explainshell.com](https://explainshell.com/)** - Understand complex commands
- **[tldr pages](https://tldr.sh/)** - Quick command reference
- **VS Code** - Great editor with shell script extensions

### Online Practice

- **[Over The Wire](https://overthewire.org/wargames/)** - Bandit levels for Linux practice
- **[Bash Scripting Challenges](https://www.hackerrank.com/domains/shell)** - HackerRank exercises

### Books

- "The Linux Command Line" by William Shotts
- "Bash Cookbook" by Carl Albing
- "Classic Shell Scripting" by Arnold Robbins

---

## Project Structure

```
shell-scripting-mastery/
├── README.md                          # This file
├── curriculum.md                       # Condensed curriculum overview
├── course_material/
│   ├── Unit_1_CLI_Fundamentals.md
│   ├── Unit_2_Shell_Scripting_Basics.md
│   ├── Unit_3_Control_Flow.md
│   ├── Unit_4_Functions_and_Organization.md
│   ├── Unit_5_IO_and_Text_Processing.md
│   ├── Unit_6_Strings_and_Regex.md
│   ├── Unit_7_Advanced_Scripting_Topics.md
│   ├── Unit_8_Package_Managers_and_System_Admin.md
│   ├── Unit_9_Practical_Projects.md
│   ├── Unit_10_Best_Practices_and_Advanced_Techniques.md
│   └── Reference_Syntax_Commands_PackageManagement.md
└── examples/                           # (Coming soon) Code examples
```

---

## Contributing

This is an open-source educational project. Contributions are welcome!

### How to Contribute

1. **Fork the repository**
2. **Create a feature branch** (`git checkout -b feature/improvement`)
3. **Make your changes**
4. **Test thoroughly**
5. **Submit a pull request**

### Contribution Ideas

- Fix typos or improve explanations
- Add more examples
- Create additional exercises
- Translate to other languages
- Add video tutorials

---

## License

This course is released under the **MIT License**. You are free to:
- Use this material for personal learning
- Share with others
- Modify and adapt
- Use in commercial training (with attribution)

See [LICENSE](LICENSE) file for details.

---

## Support

If you find this course helpful, please:
- ⭐ **Star this repository**
- 🐛 **Report issues** you encounter
- 💡 **Suggest improvements**
- 📢 **Share with others**

---

## Acknowledgments

This course was created to make shell scripting accessible to everyone. Special thanks to:
- The open-source community
- Contributors and reviewers
- Students who provided feedback

---

## Contact

- **Issues:** [GitHub Issues](https://github.com/yourusername/shell-scripting-mastery/issues)
- **Discussions:** [GitHub Discussions](https://github.com/yourusername/shell-scripting-mastery/discussions)

---

## Quick Navigation

**📚 [Start Learning →](course_material/Unit_1_CLI_Fundamentals.md)**

**📋 [View Full Curriculum](curriculum.md)**

**🔍 [Reference Guide](course_material/Reference_Syntax_Commands_PackageManagement.md)**

---

<p align="center">
  <strong>Happy Scripting! 🚀</strong>
</p>

<p align="center">
  Made with ❤️ for the open-source community
</p>
