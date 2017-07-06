---
hash-prefix: '25c9523d_'
menu:
- class: 'documentation button button-green align-right'
  href: '/doc'
  title: Documentation
title: Architecture of LugBench
---

* This will become a table of contents (this text will be scraped).
{:toc}

## Homepage

The homepage is located at `http://lugbench_url/gpus`.

## Project architecture

### Configuration

The project contains some configuration files.
Here is the list.

<table>
<colgroup>
<col width="16%" />
<col width="83%" />
</colgroup>
<thead>
<tr class="header">
<th>Files</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>package.json</td>
<td>The definition of dependencies, used by <code>npm</code> when installing the project.</td>
</tr>
<tr class="even">
<td>gulpfile.js</td>
<td>Configuration of different Gulp tasks.</td>
</tr>
<tr class="odd">
<td>tsconfig.json</td>
<td>TypeScript configuration file.</td>
</tr>
<tr class="even">
<td>tslint.json</td>
<td>TypeScript linter configuration file.</td>
</tr>
<tr class="odd">
<td>conf/*.js</td>
<td>Configuration of additional modules used by the project.</td>
</tr>
</tbody>
</table>

### Sources

All the sources files are located in the `src` folder.
The initialization page is located at the root of this `src` folder.
Then, all the components and models are located in the `src/app` folder.
