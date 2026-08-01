randi() % 6 + 1          # random int 1–6, dice roll style
randi_range(1, 6)          # cleaner way to do the same thing
randf()                      # random float 0.0–1.0
randf_range(1.5, 4.5)          # random float in a range

var deck = [1, 2, 3, 4, 5]
deck.shuffle()                 # shuffles an array in place
deck.pick_random()               # picks one random element

randomize()                        # seeds from OS entropy — call once at game start
seed(12345)                          # or seed manually for reproducible runs, like Python's random.seed()