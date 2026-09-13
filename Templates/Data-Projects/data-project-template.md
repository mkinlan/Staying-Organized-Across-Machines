# Data Project Template

Here is an example project structure for use in any IDE. 
```
data-project-template/
├── README.md
├── .gitignore
├── data/
│   ├── raw/
│   ├── interim/
│   └── processed/
├── notebooks/
├── src/
├── outputs/
│   ├── figures/
│   └── tables/
├── docs/
│   ├── data_dictionary.md
│   ├── decisions.md
│   └── project_notes.md
└── tests/
```
## What I like about it
<b> Build in the docs</b> - My favorite thing about this layout is the <b>docs folder</b>, especially the data_dictionary.md file. How many times have you started looking at data and thought, "Why is there no data dictionary????" I know I've thought this, but have not be the greatest at actually creating them either. Having a documentation folder as part of a standard layout puts it right in your face - less easy to forgot that you need to document!

<b>"Final" vs Client-Ready</b> - This system adds an additional layer of nuance to outputs. Final code output lives in the 'processed' subfolder, while formatted and presentation-ready data tables live in the 'outputs' subfolder. This is helpful for times when you need the final version of your dataframe to use in R or Python, but your customer wants that same data in Excel with branded colors and columns that contain underscores. This way you can keep track of "What did I actually give them?"

<b>Notes!</b> - Notes go in two places here:
1) The notebooks subfolder for coding notes (think Quarto, RMarkdown, Jupyter, etc.)
2) The docs/ subfolder for markdown or text notes (think TextEdit, Word Doc, 'What's my next step in this project?' notes). Next to the README doc, this would be stop #2 for a co-worker that needs to pick up where you left off, or reproduce your work. 

