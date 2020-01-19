---
title: Property based testing
summary: >-
  This is the story about how I found a bug with Property based testing while
  still not sure how to best utilize it.
date: 2020-01-06T18:30:31.937Z
draft: true
---
-- Maybe mention some other names of it here too
I have read a bit about and watched some talks about Property based testing and I have been really fascinated about it but only seen some expamles that shows math properties like addition and subtraction. Now when I had my first rather large own project that I had put a bit of time into it seemed like a perfect time to try it out for real.

The project I mention is the drinking game [Giving Game](https://giving-game.se) which is built using Elm in the frontend and Elixir with Phoenix and Channels (abstractions over Web Sockets). It is a turn based card game where you can put card in your hand and play the cards on other players. Every action a player makes in the game is a command that gets sent to the Elixir backend. I started by trying to create a Property test for the frontend using the [elm-test](https://package.elm-lang.org/packages/elm-explorations/test/latest) library which does a good work explaining strategies on how to write good tests. But I could not find any good place where it would fit so instead I looked at the backend and the library [Stream data](https://hexdocs.pm/stream_data/StreamData.html)([Release notes StreamData](https://elixir-lang.org/blog/2017/10/31/stream-data-property-based-testing-and-data-generation-for-elixir/))

As it is a game the only state is the game state and one thing that I have not been doing a very good job with in the backend is to [make impossible states unrepresentable](https://www.youtube.com/watch?v=IcgmSRJHu_8) as suggested by the elm-test library. What I instead had done was making each command from the fronted be a pure function that receives the game state, and some extra data and produces a new game state. This made it possible to create some code similar to this:

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
So what happens here? A list of all possible commands in the game are created - command_list - then a game with two players are setup and started. After that every command is applied on the game and lastly there are some validators to check that the game is still in a valid state.

This code in turn helped me find a bug that was in the game and would have been really hard to find on its own and as a reference when running this test there was at least 30 successful runs every time before the test failed. The assert that was failing was the "with_more_than_20_points_it_should_be_end_game". The reason this happened was that at the time there were two ways to get points in the game, through a card type called Give card and a type called Chance card and only Give card implemented a check if the game reached 20 points so it could be put in the end game state. And as you can guess the Chance cards does not give points every time so its pretty hard to win with that card. So finding this bug and reproducing it manually would have been really hard. Now it was easy to find and then the fix was only to move the check to after every time a player have been picked for a card.



During the same time I created this test I had another really annoying bug that crashed the game every once in a while and it was really hard to find out what was happening. So to fix it a used the Elixir Supervisor mindset where you crash and recover when something is bad. So I started to save the state after every command was run and if something went wrong I just read in the good old state. So I though everything was fine until it chashed again. Could not believe my eyes. But thanks to the property based tests I saw a lot of nil values generated in an array with cards marked as done which caught my attention.

