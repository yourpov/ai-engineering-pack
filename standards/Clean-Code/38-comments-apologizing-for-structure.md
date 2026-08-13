# 38. When comments apologize for structure, change the structure

> **Rule:** Closing-brace comments and banner section markers are symptoms. Fix size and modularity instead.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

`// end if user` and `// ===== AUTH =====` try to make a mess navigable. Small functions and real modules remove the need. Rare framework boilerplate may justify a marker — overuse becomes noise.

---

## Rules

- No closing-brace comments — shorten the block.
- No banner farms — split files/types by responsibility.
- Markers only for unavoidable framework sections, sparingly.
- If you need a map of the file, the file is too big.

---

## Bad vs good

### Bad
```text
} // end for
} // end if
// ******** UTILITIES ********
```

### Good
```text
// short functions; separate modules for auth, db, files
```

---

## Review checklist

- [ ] Are comments compensating for length or nesting?
- [ ] Would a split delete the banners?

---

## Related standards

- 15 One thing
- 26 Extract
- 19 Noise

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
