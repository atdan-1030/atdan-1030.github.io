---
layout: post
title: Scraping HW
image: /img/hello_world.jpeg
---

## used wget to download html from http://www.perseus.tufts.edu/hopper/collection?collection=Perseus:collection:RichTimes

this downloaded a really messy html file into my folder

## found a regex to extract all links from the txt-file opening it in sublime text. 
1. i typed in a regex like doc=Perseus%3atext%3a2006.05.0001 and simply replaced the end with for dots .... in order to include all relevant links
2. then i pressed alt+enter to select all links and copy-pasted them

## copied the matches in a new txt-file, which i opened with sublime text

1. since in the html-file i could only find parts of the links because assumably they are originally located inside a folder and work as internal references, so 
2. i had to replace the beginning of the links with the real URL found in the internet

## using a regex replacement 

find: ^text
replace: http://www.perseus.tufts.edu/hopper/**dl**text

the dl was important - it was the html-version of the texts

### thats it - the summary makes it look easier then it was for i missed the classed two weeks ago