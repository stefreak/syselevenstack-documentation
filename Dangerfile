# Sometimes it's a README fix, or something like that - which isn't relevant for
# including in a project's CHANGELOG for example
declared_trivial = github.pr_title.include? "#trivial"

# Make it more obvious that a PR is a work in progress and shouldn't be merged yet
warn("PR is classed as Work in Progress") if github.pr_title.include? "[WIP]"

# Warn when there is a big PR
warn("Big PR") if git.lines_of_code > 500

prose.ignored_words = [
"TOC", # mkdocs placeholder for table of contents

# brands
"SysEleven",
"OpenStack",
"CoreOS",
"S3",
"NGINX",

# flavors
"M1",
"Micro",
"Small",
"Medium",
"Large",
"m1.small",
"m1.micro",
"m1.medium",
"m1.large",

# network protocols
"DHCP",
"ICMP",
"IP",
"IPv4",

# abbreviations
"SDKs", # Software Development Kit
"UID", # Unique Identifier
"YAML" # YAML Ain't Markup Language
]

prose.lint_files "*/**/*.md"

# TODO: spell check for german
prose.check_spelling "english/**/*.md"
