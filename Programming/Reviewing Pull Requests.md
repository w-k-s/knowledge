# Reviewing Pull Requests

When I review a PR, I basically look for 3 things:

1. Immutability
2. Proper Error Handling
3. Simplicity

## Immutability

I first read about immutability from "Effective Java" by Joshua Bloch. This was right after I graduated and I'd found work as a Java Developer.

At first, I had my reservations about writing code in an immutable way: I think my main concern was that it would require frequent memory allocations, which didn't seem worth the benefit. 

However, the book put the concept of immutability in my mind and what happened was that time and time again, I would investigate a bug at work, figure out what the problem was and then realise: "Huh, if our code had been immutable, this bug wouldn't even be possible".

That made me take my first leap into learning to right immutable code. I fell in love with it very quickly because it meant that your objects could only transition between fixed and known states. This made it very easy to understand what was going on. It felt like a lot of clutter had been removed from my head.

I look for immutability in PRs because I feel that it eliminates a large class of errors, saving time in debugging and resulting in more predictable code.

## Proper Error Handling

I studied programming on my own, watching YouTube tutorials and reading language documentation.

What I love about programming is that when you do something wrong, you get a complete list of everything that was going on when the application crashed and there, right near the very top is an explanation of the problem along with the line number and file that caused it. How cool is that!

We programmers are uniquely lucky in that we can return error details when something goes wrong. If other professions knew about this, they'd be envious! I mean can you imagine what it would be like for surgeons if, when they cut open a human body, they'd see an arrow pointing directly where the problem is and explaining exactly what they need to do to fix it! 

And yet, despite our ability to return error details, most programmers don't bother with it. We are so focused on developing functionality that propagating error messages feels like such a chore. In the end, we pay the price by spending hours debugging an issue that could have been solved in seconds if we'd done a better job logging error details.

I take error handling very seriously. If I'm writing the code to describe what to do when something goes wrong (e.g. in the `catch` block), I want to give myself as much information as I can so that I can recognise the problem immediately and start working on the solution. 

I make life easier for myself and I expect other developers to do the same. So ensuring that errors are handled and the complete context of the error is logged is something that I pay close attention when reviewing PRs.

## Simplicity

Simplicity when it comes to code means the code is easy to follow and, by extension, easy to debug and easy to extend. 

I evaluate that the code is simple by asking myself: 

**Do I understand what it is doing well enough to be able to explain it to someone else?**

- If not, is it because the code is complex or complicated?
- If it is complex, then I will work with the developer to simplify it. 
- If it is complicated, I will ask the developer to document it.

**Does the code have a nice flow** 

Code has a nice flow when you have a `main` method that gives you a story-like outline of what needs to happen.

You can then drill down into the methods called in this `main` method to understand how specific parts of the logic are achieved. I'm not sure if I explained that well so here's a contrived example: 

```java
public class TicTacToeController{

	private TicTacToeView view;
	private TicTacToe game;

	public TicTacToeController(int numberOfPlayers){
		view = new TicTacToeView();
		game = new TicTacToe(numberOfPlayers);
	}

	public void start(){
		while(!game.isOver()){
			if(!game.addMove(nextMove())){
				view.showError("Cell is already occupied");
				continue;
			}
			updateGrid();
		}
		view.showWinner(game.winner());
	}

	private Move nextMove(){
		Player player = game.nextPlayer();
		return switch(player){
			case Player.ONE, Player.TWO -> view.promptMove(player);
			case Player.AI -> game.generateMove();
		}
	}

	private void updateGrid(){
		final Mark[] grid = game.grid();
		for(int i=0; i<grid.length; i++){
			view.mark(i, grid[i]);
		}
		view.redraw();
	}
}
```

## Conclusion

In this article, I wanted to document the 3 things that matter the most to me when I review a PR.
This is not to say that these are the 3 things that every developer should look at when they review PRs.

On the contrary, I think every developer should identify the things that matter the most to them when the review someone elses code.

The greatest benefit can be derived from the PR process when different developers with different concerns share their feedback: it enables the code to be improved from a multitude of dimensions.