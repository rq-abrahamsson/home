---
title: How I used Property Based Testing in one of my projects
summary: >-
  This is the story about how I found a bug - or two - with Property Based Testing while
  still not sure how to best utilize it.
date: 2020-01-06T18:30:31.937Z
draft: true
---

Many of you might ask - what is property based testing? - and i will give a short introduction but if you want to learn more I would recommend reading [this article](https://fsharpforfunandprofit.com/posts/property-based-testing/) by [Scott Wlaschin](https://twitter.com/ScottWlaschin) that have written a lot of good articles about F#, Functional Programming and Domain Modeling.

## So what is Property Based Testing?
Some other names you could hear that are are more or less the same thing are Fuzz-test, Quickcheck. So what are the properties you might ask, a really good example of properties you can find in math and more specific addition and you can see an [example using F#](https://fsharpforfunandprofit.com/posts/property-based-testing/#using-fscheck-to-test-the-addition-properties) and my attempt create a test for addition in Elixir:
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
So what you can see here is `StreamData.integer()` which is called a Fuzzer and that generates the input to the property test. Then the generated variables is used to test the properties and if a test with the generated numbers fail you will see what input was used to trigger that fail. Usually you have a more complex input like a list to your test and then it could be possible to get really large data that is hard to reason about which is why you usually would like to create something called a Shrinker. The Shrinker is used as a way to try and decrease the size and find the minimum sized input that still triggers an error.

## How I added Property Based testing to my project
I have read a bit about and watched some talks about Property Based Testing and I have been really fascinated about it but only seen really small math/made up examples. 
For the last year I have for the first time put in a lot of time into a single project and watched grow quite large. Therefore it seemed like a perfect time to try it out for real.

### About the project
The project I mention is the drinking game [Giving Game](https://giving-game.se) which is built using Elm in the frontend and Elixir with Phoenix and Channels (abstractions over Web Sockets). It is a turn based card game where you can pick up cards, put them in your hand and play the cards on other players. Every action a player makes in the game is a command that gets sent to the Elixir backend. I started by trying to create a Property test for the frontend using the [elm-test](https://package.elm-lang.org/packages/elm-explorations/test/latest) library which does a good work explaining strategies on how to write good tests - which I really recommend reading. But I could not find any good place where it would fit so instead I looked at the backend and the library [Stream data](https://hexdocs.pm/stream_data/StreamData.html) ([Release notes StreamData](https://elixir-lang.org/blog/2017/10/31/stream-data-property-based-testing-and-data-generation-for-elixir/)) which is an Elixir implementation of Property Based Testing - which was also used in the first example.

### The code
The backend consist of a game state representing the current state of the game. One thing that I have not been doing a very good job with in the backend is to [make impossible states unrepresentable](https://www.youtube.com/watch?v=IcgmSRJHu_8) as suggested by the elm-test library. Another thing I hade done good was making each command from the fronted be a pure function that receives the game state, and some extra data and produces the next game state. This made it possible to create some code similar to this - that validates properties of the whole game:

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
This code in turn helped me find a bug that was in the game and would have been really hard to find on its own. As a reference for how hard it would have been when running this test there was at least 30 successful runs every time before the test failed. The failing assert was the "with_more_than_20_points_it_should_be_end_game". The reason this happened was that at that point in time there were two ways to get points in the game, through a card type called Give Card and a type called Chance Card. Of those types only Give Card implemented a check if someone had reached 20 points and in turn be able to put the game in an End Game state. And as might have guessed the Chance Cards does not give points every time so its pretty hard to win with that card. Therefore finding this bug and reproducing it manually would have been a bit painful. But now it was instead really easy to find the bug and what was causing it. Then the fix was only to move the check be in place where it could do the check after every time a card had affected a player.

## Finding another bug
During the same time that I created this test I had another really annoying bug that made the game crash every once in a while and I had a real hard time locating and and finding out what was happening. So to fix the problem without finding the bug I used the Elixir Supervisor mindset where you fail fast and recover when something is bad. So I started to save the game state after every command was run and if something went wrong I just read in the good old state. I felt confident that it would work I though everything was fine until it crashed again. That felt really strange and I could hardly believe it. But thanks to the Property Based Tests I saw a lot of nil values generated in an array with cards marked as done which caught my attention. So I did set up a validation for the game that did not allow nil values in that array and went looking in the only places that was touching that array.

## Summary
