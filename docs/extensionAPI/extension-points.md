---
Order: 3
Area: extensionapi
TOCTitle: Contribution Points
ContentId: 2F27A240-8E36-4CC2-973C-9A1D8069F83F
PageTitle: Visual Studio Code Extension Contribution Points - package.json
DateApproved: 8/4/2016
MetaDescription: To extend Visual Studio Code, your extension (plug-in) declares which of the various contribution points it is using in its package.json extension manifest file.
---

# Contribution Points - package.json

This document covers the various contribution points that are defined in the [`package.json` extension manifest](/docs/extensionAPI/extension-manifest.md).

* [`configuration`](/docs/extensionAPI/extension-points.md#contributesconfiguration)
* [`commands`](/docs/extensionAPI/extension-points.md#contributescommands)
* [`menus`](/docs/extensionAPI/extension-points.md#contributesmenus)
* [`keybindings`](/docs/extensionAPI/extension-points.md#contributeskeybindings)
* [`languages`](/docs/extensionAPI/extension-points.md#contributeslanguages)
* [`debuggers`](/docs/extensionAPI/extension-points.md#contributesdebuggers)
* [`grammars`](/docs/extensionAPI/extension-points.md#contributesgrammars)
* [`themes`](/docs/extensionAPI/extension-points.md#contributesthemes)
* [`snippets`](/docs/extensionAPI/extension-points.md#contributessnippets)
* [`jsonValidation`](/docs/extensionAPI/extension-points.md#contributesjsonvalidation)

## `contributes.configuration`

Contribute configuration keys that will be exposed to the user. The user will be able to set these configuration options either from User Settings or from the Workspace Settings.

When contributing configuration keys, a JSON schema describing these keys is actually contributed. This ensures the user gets great tooling support when authoring VS Code settings files.

You can read these values from your extension using `vscode.workspace.getConfiguration('myExtension')`.

### Example

```json
...
"contributes": {
	"configuration": {
		"type": "object",
		"title": "TypeScript configuration",
		"properties": {
			"typescript.useCodeSnippetsOnMethodSuggest": {
				"type": "boolean",
				"default": false,
				"description": "Complete functions with their parameter signature."
			},
			"typescript.tsdk": {
				"type": ["string", "null"],
				"default": null,
				"description": "Specifies the folder path containing the tsserver and lib*.d.ts files to use."
			}
		}
	}
}
```

![configuration extension point example](images/extension-points/configuration.png)

## `contributes.commands`

Contribute an entry consisting of a title and a command to invoke to the Command Palette (`kb(workbench.action.showCommands)`).

>**Note:** When a command is invoked (from a key binding or from the Command Palette), VS Code will emit an activationEvent `onCommand:${command}`.

### Example

```json
...
"contributes": {
	"commands": [{
		"command": "extension.sayHello",
		"title": "Hello World"
	}]
}
...
```

![commands extension point example](images/extension-points/commands.png)

## `contributes.menus`

Contribute a menu item for a command to the editor or Explorer. The menu item definition contains the command that should be invoked when selected and the condition under which the item should show. The later is defined with the `when` clause which uses the key bindings [when clause contexts](/docs/customization/keybindings.md#when-clause-contexts). In addition to the mandatory `command` property, an alternative command can be defined using the `alt`-property. It will be shown and invoked when pressing `kbstyle(Alt)` while hovering over a menu item. Last, a `group`-property defines sorting and grouping of menu items. The `navigation` group is special as it will always be sorted to the top/beginning of a menu.

Currently extension writers can to contribute to:

* The Explorer context menu - `explorer/context`
* The editor context menu - `editor/context`
* The editor title menu - `editor/title`

>**Note:** When a command is invoked from a (context) menu, VS Code tries to infer the currently selected resource and passes that as a parameter when invoking the command. For instance, a menu item inside the Explorer is passed the URI of the selected resource and a menu item inside an editor is passed the URI of the document.

In addition to a title, commands can also define icons which VS Code will show in the editor menu bar.

### Example

```json
...
"contributes": {
	"menus": {
		"editor/title": [{
			"when": "resourceLangId == markdown",
			"command": "markdown.showPreview",
			"alt": "markdown.showPreviewToSide",
			"group": "navigation"
		}]
	}
}
...
```

![menus extension point example](images/extension-points/menus.png)

## `contributes.keybindings`

Contribute a key binding rule defining what command should be invoked when the user presses a key combination. See the [Key Bindings](/docs/customization/keybindings.md) topic where key bindings are explained in detail.

Contributing a key binding will cause the Default Keyboard Shortcuts to display your rule, and every UI representation of the command will now show the key binding you have added. And, of course, when the user presses the key combination the command will be invoked.

>**Note:** Because VS Code runs on Windows, Mac and Linux, where modifiers differ, you can use "key" to set the default key combination and overwrite it with a specific platform.

>**Note:** When a command is invoked (from a key binding or from the Command Palette), VS Code will emit an activationEvent `onCommand:${command}`.

### Example

Defining that `kbstyle(Ctrl+F1)` under Windows and Linux and `kbstyle(Cmd+F1)` under Mac trigger the `"extension.sayHello"` command:

```json
...
"contributes": {
	"keybindings": [{
		"command": "extension.sayHello",
		"key": "ctrl+f1",
		"mac": "cmd+f1",
		"when": "editorTextFocus"
	}]
}
...
```

![keybindings extension point example](images/extension-points/keybindings.png)

## `contributes.languages`

Contribute the definition of a language. This will introduce a new language or enrich the knowledge VS Code has about a language.

In this context, a language is basically a string identifier that is associated to a file (See `TextDocument.getLanguageId()`).

VS Code uses three hints to determine the language a file will be associated with. Each "hint" can be enriched independently:
1. the extension of the filename (`extensions` below)
2. the filename (`filenames` below)
3. the first line inside the file (`firstLine` below)

When a file is opened by the user, these three rules are applied and a language is determined. VS Code will then emit an activationEvent `onLanguage:${language}` (e.g. `onLanguage:python` for the example below)

The `aliases` property contains human readable names under which the language is known. The first item in this list will be picked as the language label (as rendered in the status bar on the right).

The `configuration` property specifies a path to the language configuration file. The path is relative to the extension folder, and is typically `./language-configuration.json`. The file uses the JSON format and can contain the following properties:

* `comments` - Defines the comment symbols
  * `blockComment` - The begin and end token used to mark a block comment. Used by the 'Toggle Block Comment' command.
  * `lineComment` - The begin token used to mark a line comment. Used by the 'Add Line Comment' command.
* `brackets` - Defines the bracket symbols that influence the indentation of code between the brackets. Used by the editor to determine or correct the new indentation level when entering a new line.
* `autoClosingPairs` - Defines the open and close symbols for the auto-close functionality. When an open symbol is entered, the editor will insert the close symbol automatically. Auto closing pairs optionally take a `notIn` parameter to deactivate a pair inside strings or comments.
* `surroundingPairs` - Defines the open and close pairs used to surround a selected string.

If your language configuration file name is or ends with `language-configuration.json`, you will get validation and editing support in VS Code.

### Example

```json
...
"contributes": {
	"languages": [{
		"id": "python",
		"extensions": [ ".py" ],
		"aliases": [ "Python", "py" ],
		"filenames": [ ... ],
		"firstLine": "^#!/.*\\bpython[0-9.-]*\\b",
		"configuration": "./language-configuration.json"
	}]
}
```
language-configuration.json
```json
{
	"comments": {
		"lineComment": "//",
		"blockComment": [ "/*", "*/" ]
	},
	"brackets": [
		["{", "}"],
		["[", "]"],
		["(", ")"]
	],
	"autoClosingPairs": [
		["{", "}"],
		["[", "]"],
		["(", ")"],
		{ "open": "'", "close": "'", "notIn": ["string", "comment"] },
		{ "open": "/**", "close": " */", "notIn": ["string"] }
	],
	"surroundingPairs": [
		["{", "}"],
		["[", "]"],
		["(", ")"],
		["<", ">"],
		["'", "'"]
	]
}
```

## `contributes.debuggers`

Contribute a 'debug adapter' to VS Code's debugger. A debug adapter integrates VS Code with a particular debug engine.
It runs in a separate process and communicates with VS Code through the VS Code debug protocol.
You must provide one (or more) executables that implement the debug adapter.

### Example

```json
...
"contributes": {
	"debuggers": [{
		"type": "node",
		"label": "Node Debug",
		"program": "./out/node/nodeDebug.js",
		"runtime": "node",
		"enableBreakpointsFor": { "languageIds": ["javascript", "javascriptreact"] },
		"initialConfigurations": [{
			...
		}],
		"configurationAttributes": {
			...
		}
	}]
}
...
```

For a full walkthrough on how to integrate a `debugger` go to [Debuggers](/docs/extensions/example-debuggers.md).

## `contributes.grammars`

Contribute a TextMate grammar to a language. You must provide the `language` this grammar applies to, the TextMate `scopeName` for the grammar and the file path.

>**Note:** The file containing the grammar can be in JSON (filenames ending in .json) or in XML plist format (all other files).

### Example

```json
...
"contributes": {
	"grammars": [{
		"language": "shellscript",
		"scopeName": "source.shell",
		"path": "./syntaxes/Shell-Unix-Bash.tmLanguage"
	}]
}
...
```

See [Adding Language Colorization](/docs/customization/colorizer.md) for instructions on using the [yo code extension generator](/docs/tools/yocode.md) to quickly package TextMate .tmLanguage files as VS Code extensions.

![grammars extension point example](images/extension-points/grammars.png)

## `contributes.themes`

Contribute a TextMate theme to VS Code. You must specify a label, whether the theme is a dark theme or a light theme (such that the rest of VS Code changes to match your theme) and the path to the file (XML plist format).

### Example

```json
"contributes": {
	"themes": [{
		"label": "Monokai",
		"uiTheme": "vs-dark",
		"path": "./themes/Monokai.tmTheme"
	}]
}
```

![themes extension point example](images/extension-points/themes.png)

See [Changing the Color Theme](/docs/customization/themes.md) for instructions on using the [yo code extension generator](/docs/tools/yocode.md) to quickly package TextMate .tmTheme files as VS Code extensions.

## `contributes.snippets`

```json
"contributes": {
	"snippets": [{
		"language": "go",
		"path": "./snippets/go.json"
	}]
}
```

## `contributes.jsonValidation`

Contributes a validation schema for a specific type of `json` file.  The `url` value can be either a local path to a schema file included in the extension or a remote server URL such as a [json schema store](http://schemastore.org/json).

```json
"contributes": {
    "jsonValidation": [{
 		"fileMatch": ".jshintrc",
 		"url": "http://json.schemastore.org/jshintrc"
	}]
}
```

## Next Steps
To learn more about VS Code extensibility model, try these topic:

* [Extension Manifest File](/docs/extensionAPI/extension-manifest.md) - VS Code package.json extension manifest file reference
* [Activation Events](/docs/extensionAPI/activation-events.md) - VS Code activation events reference

## Common Questions

Nothing yet


