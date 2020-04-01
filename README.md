#Writing Documentation
The following guide provides guidelines on how to create documentation

###Adding a new navigation menu
* Navigate to _data/sidebars/mydoc_sidebar.yml file
* Add an item (title,url,output,type) under respective Section.  for e.g.
```
  - title: Overview
    output: web, pdf
    folderitems:

    - title: Get started
      url: /index.html
      output: web, pdf
```
* The url that you added needs a corresponding markdown file.  Create the md file under pages/mydoc/

Ensure that the url ends with .html.  The md file gets generated as HTML  

###Adding a new page
* Create a new page in pages/mydoc folder as .md file
* Every page should start with the header containing title, tags, keywords, summary.  e.g.
```
---
title: Help APIs and UI tooltips
tags: [publishing, single_sourcing, content_types]
last_updated: July 3, 2016
keywords: API, content API, UI text, inline help, context-sensitive help, popovers, tooltips
summary: "You can loop through files and generate a JSON file that developers can consume like a help API. Developers can pull in values from the JSON into interface elements, styling them as popovers for user interface text, for example. The beauty of this method is that the UI text remains in the help system (or at least in a single JSON file delivered to the dev team) and isn't hard-coded into the UI."
sidebar: mydoc_sidebar
permalink: mydoc_help_api.html
folder: mydoc
---
```
* Every page should end with the footer
```
{% include links.html %}
```


###Editing an existing page
* There is a link of "Edit Me" on every page from which you can directly edit the page
* Navigate to the relevant md page in pages/mydoc directory and update it.

###Miscellaneous

####Writing a Note
* If you want to write a note, you can 
```
{% include note.html content="You can write a note like this with some links <a alt='technical writing blog' href='https://www.thoughtworks.com/'>" %}
```

####Writing a tip
```
{% include tip.html content="Content of the tip." %}
```

####Writing Code Block
* If you want to document code, you can write a block like this

```js
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.2/jquery.min.js"></script>
<script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/js/bootstrap.min.js"></script>

<script type="text/javascript">
$(document).ready(function(){
    $('[data-toggle="popover"]').popover({
        placement : 'right',
        trigger: 'hover',
        html: true
    });
</script>
```

