---
draft: true
---
```dataview
table 
	title as "Book Title",
	(pages_read / total_pages * 100) as "Completion Percentage",
	(padleft("████", floor(pages_read / total_pages * 10), "█") + padright("", 10 - floor(pages_read / total_pages * 10), "░")) as "Progress"
from #book 
where status != "To Read" and status != null
```






```dataview
table 
	title as "Book Title",
	(pages_read / total_pages * 100) as "Completion Percentage",
	(padleft("████", floor(pages_read / total_pages * 10), "█") + padright("", 10 - floor(pages_read / total_pages * 10), "░")) as "Progress"
from #book 
where status = "To Read"
```
