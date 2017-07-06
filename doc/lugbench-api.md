---
hash-prefix: '811b596f_'
menu:
- class: 'documentation button button-green align-right'
  href: '/doc'
  title: Documentation
title: LugBench API Reference
---

* This will become a table of contents (this text will be scraped).
{:toc}

# API documentation

## List of endpoints

<table>
<colgroup>
<col width="8%" />
<col width="22%" />
<col width="68%" />
</colgroup>
<thead>
<tr class="header">
<th>Method</th>
<th>Route</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>GET</td>
<td><code>/api/v1/gpus</code></td>
<td>Returns all GPUs present in the database.</td>
</tr>
<tr class="even">
<td>GET</td>
<td><code>/api/v1/gpus/:id</code></td>
<td>Returns the GPU with the id &quot;:id&quot; if present in the database.</td>
</tr>
<tr class="odd">
<td>PUT</td>
<td><code>/api/v1/gpus</code></td>
<td>Add or edit a GPU if present in the database.</td>
</tr>
</tbody>
</table>

**Note:** The details of the object to pass in the payload is available [online on the API's repository](https://github.com/Lugdunum3D/LugBench-API/blob/dev/v1/models/gpu/index.js "Mongoose Schema").
The object has to be formatted in json.

## Response codes

Here is the response codes returned by the back-end.

<table>
<colgroup>
<col width="16%" />
<col width="83%" />
</colgroup>
<thead>
<tr class="header">
<th>Response code</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>200</td>
<td>Success - Request returned without any problem.</td>
</tr>
<tr class="even">
<td>201</td>
<td>Creation success - Object inserted in the database without any problem.</td>
</tr>
<tr class="odd">
<td>400</td>
<td>Bad request - Some headers or fields are missing.</td>
</tr>
<tr class="even">
<td>500</td>
<td>Server error - Please open an issue or contact us.</td>
</tr>
</tbody>
</table>

# Unit tests

Our API is covered by unit tests. We will use [Mocha](https://mochajs.org/), a feature-rich JavaScript testing framework running on Node.js.
All creation and retrieving of data are tested.
