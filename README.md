# Programming with Python I

**Programming with Python I** is a practical, session-based introduction to Python programming. The course starts with core language skills and builds toward applications involving APIs, web frameworks, data analysis, and interactive dashboards.

Materials are organized by session. Most sessions include a Markdown concepts guide and a Jupyter notebook with exercises. The notebooks are intended for active practice: read the guide, work through the unsolved exercises, and run the code as you go.

## Learning outcomes

By the end of the course, learners will be able to:

- Write Python programs using variables, control flow, functions, and built-in data structures.
- Handle invalid input, work with files, and clean text and structured data.
- Organize code into reusable modules and classes.
- Use Python libraries to access APIs, build web applications, analyze data, and create dashboards.
- Read, test, and improve code with an emphasis on clarity and maintainability.

## Course outline

| Session | Subject | Concepts guide | Exercises |
| --- | --- | --- | --- |
| 1 | Python foundations and project setup | [Guide](session-01-intro-and-software/session_1_concepts_guide.md) | [Notebook](session-01-intro-and-software/session_1_exercises.ipynb) |
| 2 | Data types, control flow, and error handling | [Guide](session-02-data-types-control-flows-error-handling/session_2_concepts_guide.md) | [Notebook](session-02-data-types-control-flows-error-handling/session_2_exercises.ipynb) |
| 3 | Advanced data structures | [Guide](session-03-advanced-data-structures/session_3_concepts_guide.md) | [Notebook](session-03-advanced-data-structures/session_3_exercises.ipynb) |
| 4 | Python idioms and patterns | [Guide](session-04-python-idioms-and-patterns/session_4_concepts_guide.md) | [Notebook](session-04-python-idioms-and-patterns/session_4_exercises.ipynb) |
| 5 | File I/O and structured data | [Guide](session-05-file-io-and-structured-data/session_5_concepts_guide.md) | [Notebook](session-05-file-io-and-structured-data/session_5_exercises.ipynb) |
| 6 | Advanced strings and data cleaning | [Guide](session-06-advanced-string-and-data-cleaning/session_6_concepts_guide.md) | [Notebook](session-06-advanced-string-and-data-cleaning/session_6_exercises.ipynb) |
| 7 | Command-line tools and automation | [Guide](session-07-command-line-tools-and-automation/session_7_concepts_guide.md) | [Notebook](session-07-command-line-tools-and-automation/session_7_exercises.ipynb) |
| 8 | Object-oriented programming foundations | [Guide](session-08-oop-foundations/session_8_concepts_guide.md) | [Notebook](session-08-oop-foundations/session_8_exercises.ipynb) |
| 9 | Advanced object-oriented programming | [Guide](session-09-advanced_oop-techniques/session_9_concepts_guide.md) | [Notebook](session-09-advanced_oop-techniques/session_9_exercises.ipynb) |
| 10 | Interactive personal budget manager | [Guide](session-10-interactive-budget-manager/session_10_concepts_guide.md) | [Notebook](session-10-interactive-budget-manager/session_10_exercises.ipynb) |
| 11 | Foundations review | [Guide](session-11-stop-and-review/session_11_concepts_guide.md) | [Notebook](session-11-stop-and-review/session_11_exercises.ipynb) |
| 12 | APIs and JSON data | [Guide](session-12-apis-and-json-data/session_12_concepts_guide.md) | [Notebook](session-12-apis-and-json-data/session_12_exercises.ipynb) |
| 13 | Organizing code with modules | [Guide](session-13-organizing-code-with-modules/session_13_concepts_guide.md) | [Notebook](session-13-organizing-code-with-modules/session_13_exercises.ipynb) |
| 14 | Popular Python libraries | [Guide](session-14-popular-python-libraries/session_14_concepts_guide.md) | [Notebook](session-14-popular-python-libraries/session_14_exercises.ipynb) |
| 15 | Web development with Flask | [Guide](session-15-web-development-with-flask/session_15_concepts_guide.md) | [Notebook](session-15-web-development-with-flask/session_15_exercises.ipynb) |
| 16 | APIs with FastAPI | [Guide](session-16-apis-with-fastapi/session_16_concepts_guide.md) | [Notebook](session-16-apis-with-fastapi/session_16_exercises.ipynb) |
| 17 | Data analysis with pandas and DuckDB | [Guide](session-17-data-analysis/session_17_concepts_guide.md) | [Notebook](session-17-data-analysis/session_17_exercises.ipynb) |
| 18 | Interactive dashboards with Streamlit | [Guide](session-18-interactive-dashboards/session_18_concepts_guide.md) | [Notebook](session-18-interactive-dashboards/session_18_exercises.ipynb) |
| 19 | Data explorer and visualizer | [Guide](session-19-data-explorer-and-visualizer/session_19_concepts_guide.md) | [Notebook](session-19-data-explorer-and-visualizer/session_19_exercises.ipynb) |

## Getting started

### Requirements

- Python 3.14, as specified by the project configuration.
- [uv](https://docs.astral.sh/uv/) for managing the project environment and dependencies.
- Jupyter support, included in the project's declared dependencies. Visual Studio Code with the Python and Jupyter extensions is also a convenient way to work with the notebooks.

### Set up the environment

From the repository root, install the declared dependencies without installing the repository itself as a Python package:

```bash
uv sync --no-install-project
```

Open a session's `.ipynb` notebook in Jupyter or Visual Studio Code and run the cells in order. To start Jupyter from the project environment:

```bash
uv run --no-sync jupyter lab
```

Some later sessions use additional libraries or services. Install any session-specific dependencies as directed in that session's guide before running its exercises. API exercises may require an internet connection; Flask, FastAPI, and Streamlit exercises include their own run instructions.

## How to use the materials

1. Start with the concepts guide for the session.
2. Read the explanations alongside the code examples.
3. Complete the unsolved exercise notebook, running cells in order.
4. Review the output and try changing inputs to explore edge cases.

Some session folders also contain solved notebooks. Their availability varies; use the unsolved notebook for practice first.

## Repository structure

```text
.
├── session-01-.../
│   ├── session_1_concepts_guide.md
│   └── session_1_exercises.ipynb
├── ...
├── session-19-.../
├── LICENSE
├── pyproject.toml
└── uv.lock
```

The project configuration and lock file describe the shared Python environment. Some tools are specific to individual sessions and should be added only when those materials require them.

## License

This repository is distributed under the [MIT License](LICENSE).
