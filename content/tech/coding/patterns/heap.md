---
lastSync: Wed Jul 09 2025 23:44:20 GMT+0530 (India Standard Time)
draft: true
---
### Python Library for Heap

`Heapq` is implemented as min-heap where smallest element has highest priority

```
import heapq

pq = []

# push
heapq.heappush(pq, val)

# pop
heapq.heappop(pq)

# heapify
heapq.heapify(pq)
```

For implementing max heap with `heapq` insert inverted `val`,

```
heapq.heappush(pq, -val)
```

#### Object based heap

Creating a min heap out of python objects.

```
class AnyClass():
	def __init__(self, params: Any):
		self.params = params

	def __lt__(self, other):
		# Add comparator logic here


pq = [AnyClass(params=1), AnyClass(params=1), AnyClass(params=1), ...]

heapq.heapify(pq)
```
### Python implementation for heap functions

#### heapify
#### pop
#### push

### Usage

1. Scheduling tasks based on priority
2. 
