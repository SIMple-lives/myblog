---
photo: https://apps.kde.org/app-icons/org.kde.cantor.svg
title: Integrate KTextEditor into Cantor(2)
date: 2025-08-22
tag:
- gsoc
- KDE
category: 
- notes
- c++
- Qt
---

# Integrate KTextEditor into Cantor(2)

Over the past few months, I’ve been working on an important refactor in Cantor: **migrating the editor for command entries** from our in-house `QTextDocument` implementation to the powerful **KTextEditor** framework. In my previous update, I described how Phase 1 laid the foundation—command cells were migrated to use `KTextEditor::View`, enabling basic syntax highlighting and a modern editing experience.

Today, I’m excited to share that **Phase 2 is now complete**! With this milestone, the migration of command entries to KTextEditor is fully in place, ensuring that all existing functionality works smoothly without regressions. This achievement provides a solid foundation for future enhancements while keeping Cantor stable and reliable for everyday use.

## What’s New in Phase 2

With Phase 2 now complete, command entries are fully integrated into KTextEditor. Along the way, we introduced three major upgrades to Cantor’s core architecture, paving the way for a more consistent, powerful, and future-ready user experience.

### 🔹 Unified Highlighting Framework

All syntax highlighting in Cantor is now powered by **KSyntaxHighlighting**, the same robust engine behind Kate and KWrite. This change ensures that every backend (such as Python, Maxima, R, Octave, etc.) benefits from a **consistent, accurate, and highly reliable highlighting system**.

Previously, each backend shipped with its own ad-hoc rules that were difficult to maintain and often inconsistent in style. With the new centralized approach, Cantor handles highlighting uniformly, not only providing a smoother user experience but also laying the groundwork for future support of **custom themes and user-defined keywords**.

------

### 🔹 Unified Completion Infrastructure

Code completion has likewise been consolidated into a **common framework coordinated through KTextEditor**. In the past, each backend had its own incomplete and sometimes inconsistent completion logic. Now, all completion requests are handled in a **unified, predictable manner**, with backend-specific intelligent suggestions seamlessly integrated.

The result is less duplicated code, easier maintenance, and—most importantly—a more **cohesive user experience**. Whether you are writing Python scripts or Maxima formulas, code completion now behaves consistently, making Cantor feel smarter and more reliable.

------

### 🔹 Reduced Code Redundancy

By adopting KTextEditor as the core for command entry editing, we eliminated a significant amount of **custom code that had been written in Cantor over the years to handle code completion and highlighting for the different supported languages**.

This streamlining improves maintainability, reduces potential bug risks, and makes Cantor’s codebase **more approachable for new contributors**. Developers no longer need to reimplement low-level editor features, allowing them to focus on advancing **high-level functionality**. In short: **less boilerplate, more room for innovation**.

------

## Functional demonstration: new and old comparison, take a look!

Thanks to the new **KSyntaxHighlighting** backend, we can now temporarily change the theme of command entries, demonstrating future possibilities.

Please note that this is currently a preview feature; global "sheet themes" (applying themes uniformly to the entire sheet,) are our next steps.

* **`Breeze Dark`**

  ![](https://pub-a7510641c4c0427886fce394cb093861.r2.dev/Breeze%20Dark.png)

* **`Github Dark`**

![](https://pub-a7510641c4c0427886fce394cb093861.r2.dev/Github%20Dark.png)

* **`Breeze Light`**

![](https://pub-a7510641c4c0427886fce394cb093861.r2.dev/Breeze%20Light.png)

By integrating KTextEditor, Cantor now provides a **unified** and **reliable** code completion experience for all backends (such as Python, R, and Maxima).

![](https://pub-a7510641c4c0427886fce394cb093861.r2.dev/com.png)

Cantor also supports consistent multi-cell handling, with themes and syntax highlighting applied uniformly.

![](https://pub-a7510641c4c0427886fce394cb093861.r2.dev/multi.png)

## Why This Matters

This migration is not just a technical change under the hood—it directly impacts how Cantor evolves:

- **Stability first**: by ensuring no regressions during the migration, users can continue to rely on Cantor for daily work without disruption.
- **Consistency across backends**: highlighting and completion now feel the same, no matter which computational engine you choose.
- **Future-proof foundation**: less redundant code and more reliance on KDE Frameworks means Cantor can keep pace with new features in KTextEditor and the broader KDE ecosystem.

------

## What’s Next

With command entries now fully migrated, the door is open for exciting new improvements:

* **Theming support**: enabling custom color schemes and styles, giving users the ability to tailor Cantor’s appearance to their preferences.

- **Vi mode integration**: bringing modal editing from Kate into Cantor.
- **Spell checking**: powered by Sonnet, useful for Markdown and explanatory text inside worksheets.
- **Smarter backend completions**: richer suggestions, function signatures, and inline documentation.
- **Performance work**: optimizing for very large worksheets and heavy computations.

### Theming support (planned)

For now, Cantor will keep the **Default** theme, which uses the desktop palette. This preserves the familiar look and behavior.

Next, we plan to introduce a **Worksheet Theme** setting. Users will be able to:

- Stay with **Default** (desktop palette, as before), or
- Choose a theme from **KTextEditor/KSyntaxHighlighting**, just like in Kate.

The selected theme will apply consistently across the worksheet—including command entries and results—for a unified appearance. Instead of relying on hardcoded colors or the system palette, Cantor will use the color roles provided by KTextEditor and KSyntaxHighlighting.

This approach avoids performance overhead from repeatedly reading theme files, ensures instant updates when switching themes, and lays the foundation for richer customization in the future—such as clearer distinctions between prompts, results, and errors, all within a consistent global style.

------

