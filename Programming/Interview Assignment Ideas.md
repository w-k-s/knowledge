# Interview Assignment Ideas

These are ideas for tasks to include in Programming Interview Assignments. They're based on my own experiences.

## 1. Candidate must work with an API that returns a multi-dimensional array.

### Challenge

An API that I worked with returned a list of available of half-hour slots within a week. The Response looked something like this:

```json
[
	[1,1,1,1,0,0,0,0,0,0,1,1,1,1,0,0,0,0,1,1],
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
	[0,0,0,0,0,0,0,1,1,1,1,0,0,0,0,0,0,0,0,0],
	[1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],
	[0,0,0,0,0,0,0,1,1,1,1,0,0,0,0,0,0,0,0,0],
]
```
So the outer array consists of 5 inner arrays.
Each inner array represents a day of the week:

- `[0]`: Sunday
- `[1]`: Monday
- `[2]`: Tuesday
- `[3]`: Wednesday
- `[4]`: Thursday

Each inner array consists of 20 elements. 
Each element represents a half hour slot from 9AM to 6PM. 
In other words:
- `[0]`: 9 - 9:30AM
- `[1]`: 9:30 - 10AM
- `[2]`: 10 - 10:30AM
- `[3]`: 10:30 - 11AM

Each element in the inner array could contain either a `0`, meaning the slot was unavailable, or a `1`, meaning the slot was free.

So in the response above, the first unavailable slot on Sunday is from 12 - 12:30PM.

### Expectation

The app that consumed this API passed this array around all over the place, resulting in the code being very difficult to follow. 

When I joined the project, I built a `Schedule` facade that made life a lot easier!

--- 