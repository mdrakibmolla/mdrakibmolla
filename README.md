Nice — here’s a ready-to-run **GitHub profile README generator** (CLI) you can use to quickly create a polished `README.md`. It’s a single Python script that asks for inputs (or accepts flags), supports multiple projects, skills, badges, and optional social links, and writes a clean README file.

Copy this into a file named `generate_readme.py`, run `python generate_readme.py`, and follow the prompts.

# Generator (Python — interactive + CLI)

```python
#!/usr/bin/env python3
"""
generate_readme.py
Simple interactive CLI that builds a GitHub profile README.md from your inputs.
Usage:
  python generate_readme.py        # interactive mode
  python generate_readme.py --name "Noor Rakib" --bio "AI Engineer" --out README.md
"""

import argparse
import textwrap
import sys

def ask(prompt, default=""):
    if default:
        resp = input(f"{prompt} [{default}]: ").strip()
        return resp if resp else default
    else:
        return input(f"{prompt}: ").strip()

def multi_input(prompt):
    print(prompt + " (enter a blank line to finish):")
    items = []
    while True:
        s = input("- ").strip()
        if s == "":
            break
        items.append(s)
    return items

def build_badges(badges):
    if not badges:
        return ""
    lines = []
    for b in badges:
        # user may input a full markdown badge or plain text; we keep as-is
        lines.append(b)
    return " ".join(lines) + "\n\n"

def build_projects(projects):
    if not projects:
        return ""
    s = "## Featured Projects\n\n"
    for p in projects:
        # expected format: Title | short description | repo/url (optional)
        parts = [part.strip() for part in p.split("|")]
        title = parts[0]
        desc = parts[1] if len(parts) > 1 else ""
        link = parts[2] if len(parts) > 2 else ""
        if link:
            s += f"- **[{title}]({link})** — {desc}\n"
        else:
            s += f"- **{title}** — {desc}\n"
    s += "\n"
    return s

def build_social(social_pairs):
    if not social_pairs:
        return ""
    s = ""
    for pair in social_pairs:
        # format: name|url or raw url
        parts = [p.strip() for p in pair.split("|")]
        if len(parts) == 1:
            url = parts[0]
            s += f"- {url}\n"
        else:
            name, url = parts
            s += f"- [{name}]({url})\n"
    return s + "\n"

def build_skills(skills):
    if not skills:
        return ""
    s = "## Skills & Tools\n\n"
    # render as inline bullets
    s += " ".join(f"`{sk}`" for sk in skills) + "\n\n"
    return s

def build_footer(contact_line):
    if not contact_line:
        return ""
    return f"---\n\n{contact_line}\n"

def main():
    parser = argparse.ArgumentParser(description="Generate a GitHub profile README.md")
    parser.add_argument("--name", help="Your full name")
    parser.add_argument("--title", help="Short title (e.g., AI Engineer)")
    parser.add_argument("--bio", help="Very short bio")
    parser.add_argument("--location", help="Location")
    parser.add_argument("--badges", nargs="*", help="Badges as markdown strings")
    parser.add_argument("--skills", nargs="*", help="Skills or tools (space separated)")
    parser.add_argument("--projects", nargs="*", help="Projects in format 'Title | desc | url'")
    parser.add_argument("--social", nargs="*", help="Social links in format 'Name|URL' or full URL")
    parser.add_argument("--contact", help="Contact line (email or 'Reach me at ...')")
    parser.add_argument("--out", default="README.md", help="Output filename")
    parser.add_argument("--yes", action="store_true", help="Non-interactive: skip prompts if fields provided")
    args = parser.parse_args()

    # Interactive prompts if values not provided
    name = args.name or ask("Your full name")
    title = args.title or ask("Title / role (e.g., Data Scientist, AI Engineer)", "")
    bio = args.bio or ask("Short bio (1-2 lines)", "")
    location = args.location or ask("Location (city, country)", "")
    badges = args.badges or []
    if not args.yes and not badges:
        if input("Add any markdown badges? (y/N): ").strip().lower() == "y":
            badges = multi_input("Paste badge markdown lines")
    skills = args.skills or []
    if not args.yes and not skills:
        if input("Add skills/tools? (y/N): ").strip().lower() == "y":
            skills = multi_input("Enter skills/tools (one per line)")
    projects = args.projects or []
    if not args.yes and not projects:
        if input("Add featured projects? (y/N): ").strip().lower() == "y":
            print("Enter projects as: Title | short description | url (url optional)")
            projects = multi_input("Projects")
    social = args.social or []
    if not args.yes and not social:
        if input("Add social links? (y/N): ").strip().lower() == "y":
            print("Enter social links as: Name|URL   (e.g., Twitter|https://twitter.com/you)")
            social = multi_input("Social links")
    contact = args.contact or ask("Contact line (email or one-liner)", "")

    # Build README content
    header = f"# {name}\n\n" if name else ""
    subtitle = f"**{title}**\n\n" if title else ""
    location_line = f"📍 {location}\n\n" if location else ""
    badges_md = build_badges(badges)
    bio_md = f"{bio}\n\n" if bio else ""
    skills_md = build_skills(skills)
    projects_md = build_projects(projects)
    social_md = ""
    if social:
        social_md = "## Connect with me\n\n" + build_social(social)
    footer = build_footer(contact)

    # Optional: GitHub stats & top languages badges (user can paste their own or the generator can include examples)
    stats_example = (
        "<!-- GitHub stats: replace USERNAME with your GitHub username -->\n"
        "<p align=\"center\">\n"
        "  <img src=\"https://github-readme-stats.vercel.app/api?username=USERNAME&show_icons=true&theme=default\" alt=\"GitHub stats\"/>\n"
        "</p>\n\n"
    )

    content = (
        header +
        subtitle +
        badges_md +
        bio_md +
        location_line +
        skills_md +
        projects_md +
        social_md +
        stats_example +
        footer
    )

    # Write file
    out = args.out
    try:
        with open(out, "w", encoding="utf-8") as f:
            f.write(content)
        print(f"✅ README generated and saved to {out}")
    except Exception as e:
        print("✖ Failed to write file:", e, file=sys.stderr)
        sys.exit(1)

if __name__ == "__main__":
    main()
```

# Quick usage examples

1. Interactive:
   `python generate_readme.py` — follow prompts.

2. Non-interactive (partial):
   `python generate_readme.py --name "Noor Rakib" --title "AI Engineer" --bio "Building data products and ML models." --skills Python TensorFlow PyTorch --projects "CoolProject | An ML model | https://github.com/you/cool" --social "GitHub|https://github.com/yourusername" --out README.md --yes`

# Sample minimal output (what your README might look like)

```md
# Noor Rakib

**AI Engineer**

![badge-example]

Building data products and ML models.

📍 Dhaka, Bangladesh

## Skills & Tools

`Python` `TensorFlow` `PyTorch` `Docker`

## Featured Projects

- **[CoolProject](https://github.com/you/cool)** — An ML model that predicts things.

## Connect with me

- [GitHub](https://github.com/yourusername)

<!-- GitHub stats: replace USERNAME with your GitHub username -->
<p align="center">
  <img src="https://github-readme-stats.vercel.app/api?username=USERNAME&show_icons=true&theme=default" alt="GitHub stats"/>
</p>

---
Reach me at: your.email@example.com
```

# Next steps / ideas to enhance

* Auto-fetch your GitHub username and repos with the GitHub API to populate projects (would need a token).
* Offer multiple README templates (compact, developer-focused, data-science-focused).
* Make a small web UI (Flask or static HTML + JS) so non-technical users can fill fields and download `README.md`.
* Add badges auto-generation (GitHub stars, open-source contributions, Twitter follow, LinkedIn).

---

Want me to:

* convert this to a tiny web app (single-file Flask) you can run locally, or
* add a **data-science** template version (sections for Datasets, Notebooks, Papers), or
* auto-fill from your GitHub username (I'll need the username now)?

Pick one and I’ll generate it right away. 🚀
