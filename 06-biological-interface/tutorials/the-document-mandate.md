# The Documentation Mandate

To ensure the resulting documentation is universal, secure, and accessible to Linux newcomers, we will adhere to the following principles:

* **The Trellis & The Soliton:** "The Trellis" is the autonomic layer of the architecture. "The Soliton" (e.g., Zara) is the autopoietic agent. The Soliton's functions are supported and facilitated by this autonomic layer.
* **Variable-Driven Universality:** All system paths, domain names, user accounts, and text editors will be abstracted into a standard block of Environment Variables (e.g., `$DOMAIN`, `$BASE_DIR`, `$WEB_ROOT`, `$SOLITON`, `$EDITOR`). These will be documented in a single, dedicated master tiddler. Subsequent tutorials will assume this environment profile has been sourced.
* **Debian Standardization:** The architecture assumes implementation on the latest release of Debian. We will not specify Debian version numbers to ensure the documentation does not artificially age.
* **Package Management:** Standard system dependencies will be maintained in a dedicated master list tiddler. Inline installation instructions within specific modules are strictly reserved for packages requiring specialized download methods or unique configurations.
* **Agnostic Tooling:** We will use `$EDITOR /path/to/file` for all text manipulation. To prevent inline clutter, keystroke-level tutorials for specific text editors (Vim, Nano) will be provided in a supplemental reference module rather than repeated throughout the core documentation.
* **Absolute Code Completeness:** We assume the user is a Linux beginner. Every necessary terminal keystroke must be represented in a code block.
* **Modular Progression:** Modules are entirely independent and will be identified strictly by descriptive titles (e.g., "The Local Soliton Environment"), never by sequence or phase.
* **Paced Output Generation:** Complex modules will be generated across multiple conversational turns. This ensures maximum focus, thoroughness, and quality for each specific architectural component.
* **Native Markdown:** All output will be generated in standard conversation Markdown to ensure copy-paste fidelity.
* **Taxonomic Tagging:** TiddlyWiki tag taxonomy must be rigorously standardized and documented within a dedicated reference tiddler. Because tags in this architecture directly dictate systemic routing (e.g., filtering markdown files into public versus private data layers) and drive the UI structure, ad-hoc or spontaneous tagging is avoided to maintain the physical and semantic integrity of the knowledge graph.
