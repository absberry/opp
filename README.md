# OPP — Open Projection Protocol

**Application data in familiar files and folders.**

Notes, tasks, and documents usually live inside applications. To work with them in your preferred editor or process them with a script, you have to export them or learn a separate API.

OPP is a proposed open protocol that lets applications expose their data as files and accept changes back. Saving a document in an editor, for example, can become an operation on application data.

**An application describes its entities through a common model that any compatible client can use.** The model includes data representations, hierarchy, and allowed operations. OPP Drive is the first planned client to project this model into the filesystem.

> The project is in the design stage. This README describes the intended model. The specification, SDK, and Drive have not been implemented yet.

## What it looks like

A user connects an application to OPP Drive and sees its contents in their file manager:

```text
OPP/
└── MyApp/
    ├── Notes/
    │   ├── OPP idea.md
    │   └── Roadmap.md
    └── Projects/
        └── Website/
            └── Issues/
                └── Fix login.md
```

You can open a note in a text editor, find a line through the terminal, or process several documents with a script. These tools already work with files, so they do not need a separate integration for each application.

**The application remains the source of truth.** Files and folders represent its entities: notes, projects, and tasks. Saving a file becomes a request to change an entity. Renaming, moving, and deleting are available only where the application allows them.

This model fits knowledge bases, issue trackers, CMSs, and other applications whose data naturally maps to documents and hierarchies.

## Use cases

* **Editors.** Open a note or article in your preferred editor and send edits back by saving the file.
* **CLI and scripts.** Search application data from the terminal or process several documents with a script.
* **Automation.** Connect existing tools that read and write files to application data through Drive.
* **AI agents.** Ask an agent to read project notes and update a work plan using its existing file tools. Changes go through the application's permission checks and validation rules.

## How it works

```text
Application
    ↕
OPP server (built with the SDK or implemented independently)
    ↕ HTTPS
OPP Drive
    ↕
Files and folders
```

The project has three main parts:

* **OPP Protocol** defines a common language for stable entity IDs, hierarchy, representations, metadata, allowed operations, revisions, and change exchange.
* **OPP SDK** is a library that helps developers connect an existing application to the protocol.
* **OPP Drive** is a client that presents data in the filesystem and sends changes back to the application.

The protocol does not depend on the SDK or Drive. An application can implement it in any language, and another compatible client can take the place of Drive. The application's entity descriptions remain the same for all these clients.

## Connecting an application

The developer describes which parts of the application to expose, how to represent them as files, and what users can change. The data stays in the existing database and continues to be handled by the application's own logic.

The proposed SDK API might look like this. This is an illustration, not a working example:

```js
import { opp } from '@opp/server'

opp.entity('note', {
  list: () => notes.list(),
  read: id => notes.read(id),
  representation: { mediaType: 'text/markdown' },
  capabilities: ['read', 'write', 'delete'],
  update: (id, data) => notes.update(id, data),
  delete: id => notes.delete(id)
})
```

The developer also defines the hierarchy, content formats, and meaning of operations. A note might be represented as a Markdown file, for example, while moving a task between folders could change its status if the application defines that rule.

Drive receives this description through the protocol. It does not need to know how each application works internally.

## Core principles

* **Stable identity.** An entity keeps its ID when renamed or moved. Its file path does not define its identity.
* **Application control.** The application checks permissions and validates every operation. Being able to see a file does not automatically grant permission to edit it.
* **Changes in both directions.** Updates from the application appear in files, and supported file operations flow back to the application.
* **Explicit conflicts.** The protocol must account for data revisions and report conflicting changes rather than silently overwrite them.

The next goal is to define a minimal specification and validate it with a small application, a reference SDK, OPP Drive, and a protocol conformance test suite.

## License

Licensed under the [Apache License 2.0](LICENSE).
