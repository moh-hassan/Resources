## Memory Aids for System.CommandLine v2 RTM API

Here are comprehensive ways to remember the correct v2 RTM API patterns  including visual aids, mental models, and code patterns.

---

## 1. 📝 Quick Reference Card (Printable)


### Options
```csharp
// ✅ Alias goes in constructor, Description in initializer
var option = new Option<string>("--name", "-n")
{
    Description = "Option description",
    Recursive = true,           // Global option (v2)
    Required = true,            // v2: Required (not IsRequired)
    DefaultValueFactory = _ => "default"  // v2: not SetDefaultValue
};

// ✅ Boolean flag (no description in constructor)
var flag = new Option<bool>("--flag", "-f")
{
    Description = "Flag description"
};
```

### Arguments
```csharp
// ✅ Description goes in initializer, not constructor
var arg = new Argument<string>("argName")
{
    Description = "Argument description",
    Arity = ArgumentArity.ExactlyOne,  // Required argument
    DefaultValueFactory = _ => "default"  // v2: not SetDefaultValue
};

// ✅ Required argument (ExactlyOne arity)
var requiredArg = new Argument<string>("id")
{
    Description = "Required ID",
    Arity = ArgumentArity.ExactlyOne
};
```

### Commands & Root
```csharp
// ✅ RootCommand takes description only
var root = new RootCommand("Root description");

// ✅ Command takes name and description
var cmd = new Command("verb", "Command description");
```

### Adding Items
```csharp
// ✅ Use .Add() for everything
root.Add(option);
root.Add(command);
option.Aliases.Add("-n");  // Only if not in constructor
command.Arguments.Add(arg); // or command.Add(arg) 
```

### Handler & Execution
```csharp
// ✅ SetAction with ParseResult
cmd.SetAction(parseResult =>
{
    var value = parseResult.GetValue(option);
    var argValue = parseResult.GetValue(arg);
    return Task.CompletedTask;
});

// ✅ Parse first, then InvokeAsync
return await root.Parse(args).InvokeAsync();
```

---

## 2. 🎯 Mental Model: "The Two-Phase Pattern"

```
PHASE 1: DEFINE
  ┌─────────────────────────────────────────┐
  │  Option/Argument:                       │
  │  - Constructor = Name + Aliases         │
  │  - Initializer = Description + Settings │
  │  - DefaultValueFactory not SetDefault   │
  └─────────────────────────────────────────┘
                    ↓
PHASE 2: ASSEMBLE
  ┌─────────────────────────────────────────┐
  │  Command/Argument:                      │
  │  - RootCommand = description only       │
  │  - Command = name + description         │
  │  - .Add() all the things                │
  └─────────────────────────────────────────┘
                    ↓
PHASE 3: EXECUTE
  ┌─────────────────────────────────────────┐
  │  Handler & Parse:                       │
  │  - .SetAction(parseResult => { })      │
  │  - .GetValue<T>(symbol)                │
  │  - .Parse(args).InvokeAsync()          │
  └─────────────────────────────────────────┘
```

---

## 3. 🎨 Visual Decision Tree

```mermaid
graph TD
    A[Need an Option/Argument?] --> B{Is it an Option?}
    B -->|Yes| C[Option<T>]
    B -->|No| D[Argument<T>]
    
    C --> E{Has alias?}
    E -->|Yes| F["new Option<T>("--name", "-n")"]
    E -->|No| G["new Option<T>("--name")"]
    
    F --> H["{ Description = '...' }"]
    G --> H
    
    D --> I["new Argument<T>('name')"]
    I --> J["{ Description = '...' }"]
    
    H --> K{Need default?}
    J --> K
    K -->|Yes| L["DefaultValueFactory = _ => 'value'"]
    K -->|No| M[omit]
    
    L --> N{Is it required?}
    M --> N
    N -->|Yes| O["Arity = ArgumentArity.ExactlyOne"]
    N -->|No| P[omit]
    
    O --> Q[.Add() to command]
    P --> Q
```

---

## 4. 📋 Pattern Cheat Sheet (Before/After)

| **What You Want** | **❌ OLD API** | **✅ v2 RTM API** |
|-------------------|----------------|-------------------|
| Option with alias | `new Option<string>("--name")`<br>`option.AddAlias("-n")` | `new Option<string>("--name", "-n")` |
| Option description | `new Option<string>("--name", "desc")` | `new Option<string>("--name")`<br>`{ Description = "desc" }` |
| Global option | `IsGlobal = true` | `Recursive = true` |
| Required option | `IsRequired = true` | `Required = true` |
| Default value | `SetDefaultValue("x")` | `DefaultValueFactory = _ => "x"` |
| Argument description | `new Argument<string>("n", "desc")` | `new Argument<string>("n")`<br>`{ Description = "desc" }` |
| Required argument | `new Argument<string>("n")` (implicit) | `{ Arity = ArgumentArity.ExactlyOne }` |
| Argument default | `SetDefaultValue("x")` | `DefaultValueFactory = _ => "x"` |
| Add to command | `command.AddOption(opt)` | `command.Add(opt)` |
| Handler | `Handler = CommandHandler.Create(...)` | `SetAction(parseResult => { ... })` |
| Get values | Handler method parameters | `parseResult.GetValue<T>(symbol)` |
| Execution | `root.InvokeAsync(args)` | `root.Parse(args).InvokeAsync()` |
| RootCommand | `new RootCommand() { Name = "x" }` | `new RootCommand("description")` |

---

## 5. 🔍 "Check Your Code" Checklist

Before you save, verify:

### Options
- [ ] Option aliases in constructor `new Option<T>("--name", "-n")`
- [ ] Description in initializer `{ Description = "..." }`
- [ ] Global options use `Recursive = true`
- [ ] Required options use `Required = true`
- [ ] Default values use `DefaultValueFactory = _ => value`
- [ ] No `IsGlobal`, `IsRequired`, or `SetDefaultValue`

### Arguments
- [ ] Description in initializer `{ Description = "..." }`
- [ ] Required arguments use `Arity = ArgumentArity.ExactlyOne`
- [ ] Default values use `DefaultValueFactory = _ => value`
- [ ] No `SetDefaultValue`

### Commands
- [ ] `RootCommand` takes description only
- [ ] `Command` takes name + description
- [ ] All items added with `.Add()`

### Handler
- [ ] Uses `SetAction(parseResult => { ... })`
- [ ] Gets values with `parseResult.GetValue<T>(symbol)`
- [ ] Returns `Task.CompletedTask` or `Task<int>`

### Execution
- [ ] Uses `Parse(args).InvokeAsync()`
- [ ] Not `InvokeAsync(args)` alone

---

## 6. 🧩 Mnemonic Devices

### "AID" - Always In order, Description last
```
Option: Constructor (Aliases) → Initializer (Description)
Argument: Constructor (Name) → Initializer (Description, Arity, Default)
```

### "DR. DAD" - Defaults, Required, Description, Aliases, Description
```
D - DefaultValueFactory (not SetDefault)
R - Required (not IsRequired)
D - Description (in initializer)
A - Add (not AddOption/AddArgument)
D - Defaults (Arity defaults to ZeroOrOne)
```

### "PARSLEY" - Parse, Add, Required, SetAction, DefaultValueFactory
```
P - Parse(args).InvokeAsync()
A - Add()
R - Required
S - SetAction
L - (no L - remember)
E - Everything in initializer
Y - Yes to DefaultValueFactory
```

---

## 7. 📚 Reference Code Templates

### Basic Template
```csharp
using System.CommandLine;
using System.CommandLine.Parsing;

var root = new RootCommand("Description");
var opt = new Option<string>("--name", "-n") { Description = "..." };
var arg = new Argument<string>("arg") { Description = "..." };
var cmd = new Command("verb", "Description");

root.Add(opt);
root.Add(cmd);
cmd.Add(opt);
cmd.Add(arg);

cmd.SetAction(p => {
    var v = p.GetValue(opt);
    return Task.CompletedTask;
});

return await root.Parse(args).InvokeAsync();
```

### Common Patterns
```csharp
// Boolean flag
var flag = new Option<bool>("--flag", "-f") { Description = "..." };

// Required option with default
var opt = new Option<string>("--name", "-n")
{
    Description = "...",
    Required = true,
    DefaultValueFactory = _ => "default"
};

// Required argument
var arg = new Argument<string>("id")
{
    Description = "...",
    Arity = ArgumentArity.ExactlyOne
};

// Global option (available to all subcommands)
var global = new Option<bool>("--verbose")
{
    Description = "...",
    Recursive = true
};
```

---

## 8. 🔧 VS Code Snippet

Save as `.vscode/scl.code-snippets`:

```json
{
  "SCL v2 - Option": {
    "prefix": "sclopt",
    "body": [
      "var ${1:optionName} = new Option<${2:string}>(\"--${3:name}\", \"-${4:n}\")",
      "{",
      "    Description = \"${5:description}\",",
      "    ${6:Required = true,}",
      "    ${7:Recursive = true,}",
      "    ${8:DefaultValueFactory = _ => ${9:default}}",
      "};",
      "${10:command}.Add(${1:optionName});"
    ]
  },
  "SCL v2 - Argument": {
    "prefix": "sclarg",
    "body": [
      "var ${1:argName} = new Argument<${2:string}>(\"${3:name}\")",
      "{",
      "    Description = \"${4:description}\",",
      "    ${5:Arity = ArgumentArity.ExactlyOne,}",
      "    ${6:DefaultValueFactory = _ => ${7:default}}",
      "};",
      "${8:command}.Add(${1:argName});"
    ]
  },
  "SCL v2 - Handler": {
    "prefix": "sclhandler",
    "body": [
      "${1:command}.SetAction(parseResult =>",
      "{",
      "    var ${2:value} = parseResult.GetValue(${3:symbol});",
      "    ${4:// logic}",
      "    return Task.CompletedTask;",
      "});"
    ]
  },
  "SCL v2 - Execute": {
    "prefix": "sclexec",
    "body": [
      "return await ${1:root}.Parse(args).InvokeAsync();"
    ]
  }
}
```

---

## 9. 📖 Daily Quick Review (5-minute)

Each time you start a new session, run through this mental checklist:

1. **Options**: "Constructor has aliases, Description in initializer"
2. **Defaults**: "DefaultValueFactory, not SetDefaultValue"
3. **Required**: "Required, not IsRequired"
4. **Global**: "Recursive, not IsGlobal"
5. **Handler**: "SetAction with ParseResult, not CommandHandler"
6. **Execution**: "Parse(args).InvokeAsync(), not just InvokeAsync"
7. **Root**: "RootCommand takes description only, no Name"
8. **Arguments**: "Description in initializer, Arity for required"

---

## 10. 📌 Sticky Note Cheat Sheet

Copy this to a sticky note:

```
┌──────────────────────────────────────────────────┐
│  SCL v2 RTM - QUICK REFERENCE                    │
├──────────────────────────────────────────────────┤
│  Option: new Option<T>("--name", "-n")          │
│          { Description = "...",                 │
│            Required = true,                     │
│            Recursive = true,                    │
│            DefaultValueFactory = _ => "x" }     │
├──────────────────────────────────────────────────┤
│  Argument: new Argument<T>("name")              │
│            { Description = "...",               │
│              Arity = ArgumentArity.ExactlyOne,  │
│              DefaultValueFactory = _ => "x" }   │
├──────────────────────────────────────────────────┤
│  Command: new Command("verb", "desc")           │
│  Root: new RootCommand("desc")                  │
├──────────────────────────────────────────────────┤
│  Add: .Add() for everything                     │
│  Handler: .SetAction(parseResult => { ... })    │
│  Get: parseResult.GetValue<T>(symbol)           │
│  Execute: .Parse(args).InvokeAsync()            │
├──────────────────────────────────────────────────┤
│  REMEMBER: No IsGlobal, No IsRequired,          │
│           No SetDefaultValue, No AddOption      │
└──────────────────────────────────────────────────┘
```

---

Using these memory aids, you'll be able to recall the correct v2 RTM API patterns even in new sessions. The key is to think of the API as:

1. **Construction** (aliases in constructor)
2. **Configuration** (description and settings in initializer)
3. **Assembly** (everything uses `.Add()`)
4. **Execution** (`.Parse().InvokeAsync()` and `.SetAction()`)

