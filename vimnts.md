# NORMAL MODE
## SEARCHING

/word
n - next word
N - prev word

next word on cursor *
previous ""

u - undo 
Ctrl r - redo

## Movement

h - left l - right
j - down k - up

w - move forward a word
b - move back a word
0 - begin of current line
^ - first non blank char
$ - end of line  
% - matching bracket

f char - move to char on line
t char - move to before char one line

line numb G - move cursor to chosen line
G - end of file
gg - start of file

Ctrl e - scroll window down
Ctrl u - move cursor half way up
Ctrl d - move your cursor down half way

# INSERT MODE

i - insert before the char
a - insert after the char
A - insert at the end of a line

o - newline bellow
O - newline above

d - delete
c - change
y - yank

combine ie d$
diw - delete inside word
ciw - change inside a word
dip - delete para
dis - del sent
di" - del in quotation marks
di[ - del in brackets
# BUFFERS

:buffers - list all buffers and some info

nav
:buffer ID
:bn - next buffer
:bp
:bf - first
:bl

Ctrl ^ - alt buffer

:badd filename - create buffer
:bdelete id or name - del

:%bdelete - delete all buffers

# WINDOWS

Ctrl W s - horz split
Ctrl W v - vertical split
Ctrl W n - new file
buffer id  Ctrl W ^ - 

navigate via ctrl w and arrows

jumplist

positions are saved

:jumps - display jumps
ctrl o - prev cursor pos 
ctrl i - next cursor pos

change list
