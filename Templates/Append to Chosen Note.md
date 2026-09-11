<%*

/**

* Append the current selection to an existing note you choose via a suggester.

* - Type to filter notes; shows file path and title

* - Appends with optional timestamp + backlink to source (Daily Note)

* - Handles no selection by prompting for text

* - Optional: restrict search to certain folders

*/

  

const moment = window.moment;

  

// --- Configuration ---

//const addTimestamp = true;                // true → prepend a time string

const timestampFormat = "HH:mm";          // e.g., 14:05

const addBacklink = true;                 // true → add link back to source note

const backlinkLabel = "source";           // text label before backlink

const headingPrompt = false;              // true → prompt for a heading to append under

const defaultHeadingIfMissing = "";       // e.g., "Captured"; empty = append at end

  

// Limit which folders are searchable (leave empty [] to search whole vault)

const allowedFolders = [

  // "Projects",

  // "Meetings",

  // "Topics",

  // "Reference"

];

// ----------------------

  

// 1) Get selected text (or prompt)

let selected = tp.file.selection();

if (!selected || selected.trim().length === 0) {

  selected = await tp.system.prompt("No text selected. Paste or type the text to append:");

  if (!selected) {

    new Notice("Append canceled: no text provided.");

    return;

  }

}

  

// 2) Build a list of candidate files for the suggester

const allFiles = app.vault.getMarkdownFiles(); // markdown files only

const candidates = allFiles.filter(f => {

  if (!allowedFolders.length) return true;

  return allowedFolders.some(folder => f.path.startsWith(folder + "/"));

});

  

// Create display labels and values

const display = candidates.map(f => {

  // Show "Title — folder/subfolder" to make selection easier

  const title = f.basename;

  const folder = f.path.split("/").slice(0, -1).join("/");

  const folderLabel = folder ? ` — ${folder}` : "";

  return `${title}${folderLabel}`;

});

const values = candidates.map(f => f); // the TFile

  

// 3) Let the user choose a destination using a suggester

if (!display.length) {

  new Notice("No notes found in the allowed folders.");

  return;

}

const destFile = await tp.system.suggester(display, values, false, "Append to which note? (type to filter)");

if (!destFile) {

  new Notice("Append canceled.");

  return;

}

  

// 4) Build the snippet

const timeStr = addTimestamp ? `**${moment().format(timestampFormat)}** — ` : "";

  

// Backlink: link back to the source note (current file) using wiki link

const thisFileLink = `[[${tp.file.path(true)}]]`;

const backlink = addBacklink ? `  \n↩︎ ${backlinkLabel}: ${thisFileLink}` : "";

  

const snippet = `${timeStr}${selected.trim()}${backlink}\n`;

  

// 5) Append under a heading (optional) or at end

let heading = "";

if (headingPrompt) {

  heading = await tp.system.prompt("Append under which heading? (leave blank to append at end)");

}

  

if (heading && heading.trim().length > 0) {

  const destContent = await app.vault.read(destFile);

  const safeHeading = heading.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');

  const headingRegex = new RegExp(`^\\s{0,3}#{1,6}\\s+${safeHeading}\\s*$`, "m");

  if (!headingRegex.test(destContent)) {

    await app.vault.append(destFile, `\n\n## ${heading}\n`);

  }

  

  // Insert snippet at end of that heading section

  let updated = await app.vault.read(destFile);

  const lines = updated.split("\n");

  const hdrRegex = new RegExp(`^\\s{0,3}#{1,6}\\s+${safeHeading}\\s*$`);

  let sectionStart = -1;

  for (let i = 0; i < lines.length; i++) {

    if (hdrRegex.test(lines[i])) {

      sectionStart = i;

      break;

    }

  }

  if (sectionStart === -1) {

    // Fallback append

    await app.vault.append(destFile, `\n${snippet}`);

  } else {

    // Find next heading (or EOF)

    let insertIndex = lines.length;

    for (let j = sectionStart + 1; j < lines.length; j++) {

      if (/^\s{0,3}#{1,6}\s+/.test(lines[j])) {

        insertIndex = j;

        break;

      }

    }

    lines.splice(insertIndex, 0, snippet);

    updated = lines.join("\n");

    await app.vault.modify(destFile, updated);

  }

} else if (defaultHeadingIfMissing && defaultHeadingIfMissing.trim().length > 0) {

  const heading = defaultHeadingIfMissing.trim();

  const destContent = await app.vault.read(destFile);

  const safe = heading.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');

  const headingRegex = new RegExp(`^\\s{0,3}#{1,6}\\s+${safe}\\s*$`, "m");

  if (!headingRegex.test(destContent)) {

    await app.vault.append(destFile, `\n\n## ${heading}\n`);

  }

  await app.vault.append(destFile, `\n${snippet}`);

} else {

  await app.vault.append(destFile, `\n${snippet}`);

}

  

new Notice("Appended to: " + destFile.path);

%>

