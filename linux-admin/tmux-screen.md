## TMUX
````
TMUX

НЕ ПУТАТЬ panes и windows, разные хрени. На одном окне может быть много panes.

tmux new -s radio
tmux kill-session -t radio
Ctrl + b then d - detach
tmux ls - list sessions
tmux attach -t radio 
Ctrl + b then c - create a new window within session
^& 0 close current window
^w list windows
^, rename current window
^[0-9] select window by number
Ctrl + b then n/p - loop though windows
Ctrl + b then % - split window into multiple panes vertically
Ctrl + b then " - horizontally
Ctrl-b ← → ↑ ↓ - switch between panes
Ctrl + b then [ - enter scroll mode, PgUp, PgDown, Press q to quit scroll mode
OR
Ctrl + b then PgUp/Down - to go directly into copy mode

THIS IS INSANE!! Copy Mode. 
Входим в COPY MODE, Ctrl + b then [, затем нажимаем / AND IT WILL SEARCH THOUGHT ALL THE CONTENT IN TERMINAL
Другие биндинги
Half page down               C-d             M-Down
Half page up                 C-u             M-Up
Next page                    C-f             Page down
Previous page                C-b             Page up
Scroll down                  C-Down or C-e   C-Down
Scroll up                    C-Up or C-y     C-Up
Search again                 n               n
Search again in reverse      N               N
Search backward              ?               C-r
Search forward               /               C-s

^{ / } move the current pane left / right
^! convert pane into a window
^q then [0-0] select pane by number
^q show pane numbers
^ then Ctrl + ← → ↑ ↓ (с зажатым ctrl) ресайзить ширину и высоту панелек
^x kill current pane
````
## SCREEN
````
export TERM=xterm; screen -S rp_session
screen -S my_session
Для того чтобы отсоединиться от сессии не убивая её, можно использовать ctrl+a+d (ctrl+d убьет скрин)
screen -ls выводит список сессий
screen -r my_session перейти в открую сессию.

Ctrl + a then ? - help
Ctrl + a then c  - create window
Ctrl + a then " - list windows
Ctrl + a then S - split window horizontally
Ctrk + a then | - split the current window vertically
Ctrl + a then tab - jump between regions
Ctrl + a then X - remove the current region
Ctrl + a then TAB then Ctrl + a <number> - Activate the desired window in that region
Ctrl + a then Esc, After that, you should be able to move your cursor around using the arrow keys:
↑, ↓, PgUp, PgDn and sometimes using the mouse wheel. Return control: Q or Esc

Проблема что окна не сохраняются при детаче, можно это решить:
Ctrl-a then : написать layout save default как в виме короч. На одном из окон
Либо в конфиг ~/.screenrc добавить: 
# This line makes Detach and Re-attach without losing the regions/windows layout
layout save default
````
