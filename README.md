# Perplexity Best Practices

Persistent knowledge base of lessons learned, workflow improvements, and gotchas discovered across AI-assisted development sessions with Perplexity.

Reference these files at the start of any new conversation to avoid repeating solved problems.

## Universal Files

### Always Include

| File | Topics |
|------|--------|
| [github-integration.md](./github-integration.md) | Push-first rule, SHA handling, response truncation fix, commit conventions |
| [general.md](./general.md) | How to start sessions, efficiency tips, when to update this repo |
| [writing-style.md](./writing-style.md) | Tone, directness, what to avoid, hallucination checks |

### Include When Relevant

| File | When to Include |
|------|----------------|
| [trader-profile.md](./trader-profile.md) | Trading / Deribit / crypto strategy sessions |

## Project-Specific Files

| File | Language / Stack | App |
|------|-----------------|-----|
| [projects/android-mytranslator.md](./projects/android-mytranslator.md) | Kotlin / Android | MyTranslator — Azure Speech real-time translation |
| [projects/dotnet-redinn.md](./projects/dotnet-redinn.md) | VB.NET / .NET 8 | Red Inn Court — dynamic pricing system |

## Session Starter Templates

**Hostel / AppSheet / general dev session:**
```
Refer to my best practices repo before we start:
https://github.com/Beansz2015/perplexity-best-practices

Universal files: github-integration.md, general.md, writing-style.md
Project file: projects/[name].md
```

**Trading / Deribit session:**
```
Refer to my best practices repo before we start:
https://github.com/Beansz2015/perplexity-best-practices

Universal files: github-integration.md, general.md, writing-style.md, trader-profile.md
Project file: projects/[name].md
```

**Brand new project, no existing file:**
```
No project file yet — create projects/[language]-[appname].md as we go.
```

---
_Last updated: 2026-03-25_
