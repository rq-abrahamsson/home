---
title: How I used Property Based Testing in one of my projects
summary: >-
  This is the story about how I found a bug - or two - with Property Based
  Testing while still not sure how to best utilize it.
date: 2020-01-21T20:27:31.937Z
draft: false
---
Many of you might ask "What is property based testing?" and i will give a short introduction but if you want to learn more I would recommend reading [this article](https://fsharpforfunandprofit.com/posts/property-based-testing/) by [Scott Wlaschin](https://twitter.com/ScottWlaschin). I really recommend reading other articles he has written on that page if you would like to know more about F#, Functional Programming and Domain Modeling.

## So what is Property Based Testing?

It is a specific technique of testing your code which also goes by the names Fuzz-test and Quickcheck. So why is it called Property based testing? It is of course becasue we test properties of functions. That might not be very helpful but we can find a really good example of properties in math and more specificly addition. In the article mentioned previously you can find an [example using F#](https://fsharpforfunandprofit.com/posts/property-based-testing/#using-fscheck-to-test-the-addition-properties). Below is my attempt create tests for the same properties of addition in Elixir:

```
defmodule Addition do
  use ExUnit.Case, async: true
  use ExUnitProperties

  def add(x, y) do
    x + y
  end

  property "Order of params should not matter" do
    check all x <- StreamData.integer(),
              y <- StreamData.integer() do
      result1 = Addition.add(x, y)
      result2 = Addition.add(y, x)
      assert result1 == result2
    end
  end

  property "Add 1 twice is the same as adding 2 once" do
    check all x <- StreamData.integer() do
      result1 = x |> add(1) |> add(1)
      result2 = x |> add(2)
      assert result1 == result2
    end
  end

  property "Associative property" do
    check all x <- StreamData.integer(),
              y <- StreamData.integer(),
              z <- StreamData.integer() do
      result1 = add(x, add(y, z))
      result2 = add(add(x, y), z)
      assert result1 == result2
    end
  end
end
```

So what you can see here is `StreamData.integer()` which is a *Fuzzer* that is used to generates the input to the property test. Then the generated variables is used to test the properties and if a test with the generated numbers fail you will see what input was used to trigger that fail. Usually you have more complex inputs to your tests like nested objects and lists with a lot of data that is hard to reason about. This is why you usually would like to create something called a *Shrinker*. The *Shrinker* is used as a way to try and decrease the size and find the minimum sized input that still triggers an error which makes it easier to reason about what about the data that actually made it trigger an error.

## How I added Property Based testing to my project

I did read a about and watched some talks about Property Based Testing and not really trying it out even though I have been really fascinated about it. Also all examples I have seen have been really small math/made up examples. This really inspired me and now - when I during the the last year for the first time have put a lot of time and effort into a single project and watched grow quite large - it seemed like a perfect time to try it out for real.

### About the project

The project I mention is the drinking game [Giving Game](https://giving-game.se) which is built using Elm in the frontend and Elixir with Phoenix and Channels (abstractions over Web Sockets). It is a turn based card game where you can pick up cards, put them in your hand and play the cards on other players. Every action a player makes in the game is a command that gets sent to the Elixir backend. I started by trying to create a Property test for the frontend using the [elm-test](https://package.elm-lang.org/packages/elm-explorations/test/latest) library which does a good work explaining strategies on how to write good tests - which I really recommend reading. But I could not find any good place where it would fit so instead I looked at the backend and the library [Stream data](https://hexdocs.pm/stream_data/StreamData.html) ([Release post](https://elixir-lang.org/blog/2017/10/31/stream-data-property-based-testing-and-data-generation-for-elixir/)) which is an Elixir implementation of Property Based Testing that you can see in use in the first example.

### The code

The backend consist of a game state representing the current state of the game. One thing that I have not been doing a very good job with in the backend is to [make impossible states unrepresentable](https://www.youtube.com/watch?v=IcgmSRJHu_8) as suggested by the elm-test library. On the other hand I had done good job making each command from the fronted be a pure function that receives the game state and some extra data which in turn produces the next game state. With these properties on the code there was a need to make sure the game state could not get in a bad state. In turn by having only the pure functions operate on the game state property based testing could be used to test all the possible states that could come from those functions applied. With this I created some code similar to this - that validates some properties of the game after playing 3000 rounds:

```
property "Random commands will keep the game valid" do
    playerId1 = "hej"
    playerId2 = "hej2"

    # List of commands that can transform the game.
    command_list =
      List.flatten([
        {:pick_card_from_pile, nil},
        {:done_with_turn, nil},
        {:play_card, %{:boosted => true}},
        {:play_card, %{:boosted => false}},
        {:discard_selected, %{:playerId => playerId1}},
        {:discard_selected, %{:playerId => playerId2}},
        {:select_card_from_hand, %{:playerId => playerId1, :cardIndex => 0}},
        # ect....
      ])
    player = Player.new(playerId1)
    player2 = Player.new(playerId2)
    game = Game.new(Games.get_random_id())
    {_status, game} = Game.join(game, player)
    {_status, game} = Game.join(game, player2)

    game =
      game
      |> Game.set_player_ready_state(player.id, false)
      |> Game.set_player_ready_state(player2.id, false)

    check all commands <-
                StreamData.list_of(StreamData.member_of(command_list),
                  min_length: 3_000
                ) do
      # Thanks to the property that each command produces a new game state it is possible to run a reduce with the randomly generated commands.
      game =
        commands
        |> Enum.reduce(
          game,
          apply(String.to_existing_atom("Elixir.GivingGame.Games.Game"), command, [game])
        )
      # After all commands have been run the game should still be in a valid state.
      assert isValid?(game)
      assert all_cards_on_hand_have_extra_field?(game)
      assert only_one_player_active?(game)
      assert players_have_at_most_4_cards_on_hand?(game)
      assert with_more_than_20_points_it_should_be_end_game(game)
    end
end
```

So what happens here? A list of all possible commands in the game are created - command_list - then a game with two players are setup and started. After that every command is applied on the game using a reduce and lastly there are some validators to check that the game is still in a valid state.

### Finding the bug

This code in turn helped me find a bug that was in the game and would have been really hard to find on its own. As a reference for how hard it would have been when running this test there was at least 30 successful runs every time before the test failed. The failing assert was the "with_more_than_20_points_it_should_be_end_game". The reason this happened was that at that point in time there were two ways to get points in the game, through a card type called Give Card and a type called Chance Card. Of those types only Give Card implemented a check if someone had reached 20 points and in turn be able to put the game in an End Game state. And as might have guessed the Chance Cards does not give points every time so its pretty hard to win with that card. Therefore finding this bug and reproducing it manually would have been a bit painful. But now it was instead really easy to find the bug and what was causing it. Then the fix was only to move the check to be in a place where it could do the check after every time a card had affected a player.

## Finding another bug

During the same time that I created this test I had another really annoying bug that made the game crash every once in a while and I had a real hard time locating and and finding out what was happening. So to fix the problem without finding the bug I used the Elixir Supervisor mindset where you fail fast and recover when something bad has happened. So I started to save the game state after every command was run and if something went wrong I just read in the good old state. I felt confident that it would work I though everything was fine until it crashed again. That felt really strange and I could hardly believe it. But thanks to the Property Based Tests I got a hint on how to solve it. It was that I saw a lot of nil values generated in an array with cards marked as done which caught my attention. So I did set up a validation for the game that did not allow nil values in that array and went looking in the only places that was touching that array and how it could have started to produce nil values. I found that the only way was when the array was empty and it should not have been possible to send the command from the frontend when the array was empty. I was therefore a bit confounded until I realized that it had to be that the frontend had not got an update and still showed card from the array in the UI which made the player try to press the button again and in turn make the array generate a nil value which then made the game crash. The fix was simple but finding it was hard and would have been even harder without the Property Based Tests.

## Summary

I am not completely sure I have used the technique in a correct way by having this large state and only testing the correctness of parts of it. Also not using a shrinker is probably not a good way to move forward but I have not felt the need for one yet and have not been completely sure on how to do it. But the investment to try it out have already payed of and helped me a lot so I really recommend trying it out because it forces you to think about the testing of your application in a completely different way and what it is that your functions are really doing. I have learnt a lot but still have a lot to learn about it.

Thanks for reading this far, hope you enjoyed it!
