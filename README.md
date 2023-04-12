#Binary Bomb Lab:

This file is a write up for the Binary Bomb Lab and we are gonna start with..

##Phase 1:

You can actually find it here <https://www.youtube.com/watch?v=WxTSOpG6X5M>. So let us skip it for now and start with `phase_2`.

##Phase 2:

So, after phase 1 defused we set a break point at `phase_2` and disassemble it: 
```
                call    j_read_six_numbers
                mov     eax, 4
                imul    rax, 0
                cmp     [rbp+rax+120h+var_F8], 1
                jz      short loc_7FF76B4820FC
                call    j_explode_bomb
loc_7FF76B4820FC:                       ; CODE XREF: phase_2+65↑j
                mov     [rbp+120h+var_11C], 1
                jmp     short loc_7FF76B48210D


loc_7FF76B482105:                       ; CODE XREF: phase_2:loc_7FF76B482130↓j
                mov     eax, [rbp+120h+var_11C]
                inc     eax
                mov     [rbp+120h+var_11C], eax

loc_7FF76B48210D:                       ; CODE XREF: phase_2+73↑j
                cmp     [rbp+120h+var_11C], 6
                jge     short loc_7FF76B482132
                movsxd  rax, [rbp+120h+var_11C]
                mov     ecx, [rbp+120h+var_11C]
                dec     ecx
                movsxd  rcx, ecx
                mov     ecx, [rbp+rcx*4+120h+var_F8]
                shl     ecx, 1
                cmp     [rbp+rax*4+120h+var_F8], ecx
                jz      short loc_7FF76B482130
                call    j_explode_bomb

loc_7FF76B482130:                       ; CODE XREF: phase_2+99↑j
                jmp     short loc_7FF76B482105
```
* `phase_2 disassembly`

We could obviously see it calls a function called `read_six_numbers`, and if we disassemble it we would found that it reads six integers as input with function `sscanf`.
```
            lea     rdx, aDDDDDD    ; "%d %d %d %d %d %d"
            mov     rcx, [rbp+0F0h+Buffer] ; Buffer
            call    j_sscanf
            mov     [rbp+0F0h+var_EC], eax
            cmp     [rbp+0F0h+var_EC], 6
            jge     short loc_7FF76B48306E
            call    j_explode_bomb
loc_14001306E:                          ; CODE XREF: read_six_numbers+A7↑j
            lea     rsp, [rbp+0E8h]
            .  pop     rdi
            pop     rbp
            retn
```
* `read_six_numbers disassembly`

we can see it loads some format into `rdx` "%d" which is the format for integer number then it calls `sscanf` function which returns with the number of inputs we entered and sotre it in `eax` then compare it to 6
```
lea     rdx, aDDDDDD    ; "%d %d %d %d %d %d"       ;integer format
mov     rcx, [rbp+0F0h+Buffer] ; Buffer
call    j_sscanf
mov     [rbp+0F0h+var_EC], eax
cmp     [rbp+0F0h+var_EC], 6
jge     short loc_7FF76B48306E
```
* `input 6 numbers`

And we could see that it compares the first number with one and if not equal explode the bomb, and if equal jump to some address. So, we start with the set 1 2 3 4 5 6 and walk through the assembly………
```
cmp     [rbp+rax+120h+var_F8], 1
jz      short loc_7FF76B4820FC
call    j_explode_bomb
```
* `If first number is one or not`

we see a sequence for a counter and a loop for checking…
We could see that the condition for the loop is the counter below 6. Inside the loop, it does a shift lift operation with 1 `multiply with 2` to the numbers in the required sequence and starts with 1 and then compare it with each number in our input respectively after every shift operation. In other hand, it means that we shift lift 1 five times, and the result form each shift operation is an input, so shl 1 `first number in the input` is 2 `the second` number, shl 2 is 4 `the third number` and so on.
```
loc_7FF76B4820FC:
                mov     [rbp+120h+var_11C], 1       ;counter
                jmp     short loc_7FF76B48210D
loc_7FF76B482105:
                mov     eax, [rbp+120h+var_11C]
                inc     eax
                mov     [rbp+120h+var_11C], eax
loc_7FF76B48210D:
                cmp     [rbp+120h+var_11C], 6
                jge     short loc_7FF76B482132      ;End phase_3
                movsxd  rax, [rbp+120h+var_11C]
                mov     ecx, [rbp+120h+var_11C]
                dec     ecx
                movsxd  rcx, ecx
                mov     ecx, [rbp+rcx*4+120h+var_F8]
                shl     ecx, 1
                cmp     [rbp+rax*4+120h+var_F8], ecx
                jz      short loc_7FF76B482130
                call    j_explode_bomb
loc_7FF76B482130:
                jmp     short loc_7FF76B482105 ;Back to the counter block
```
* `shifting loop`

So, from here we could obviously figure out that the sequence required is `1 2 4 8 16 32`.

And yeah, it works😊
![result_Phase_2](/Bomb%20Lab/Phase_2%20defused.png)
* `phase_2 defused`


##Phase 3:

We move the breakpoint to `phase_3` and continue through the Bomb. Disassembling it...
```
                lea     r9, [rbp+150h+var_12C]      ;second input
                lea     r8, [rbp+150h+var_14C]      ;first input
                lea     rdx, aDD        ; "%d %d"
                mov     rcx, [rbp+150h+Buffer] ; Buffer
                call    j_sscanf
                mov     [rbp+150h+var_EC], eax
                cmp     [rbp+150h+var_EC], 2
                jge     short loc_7FF76B48220E
loc_7FF76B48220E:                       ; CODE XREF: phase_3+77↑j
                mov     eax, [rbp+150h+var_14C]
                mov     [rbp+150h+var_1C], eax
                cmp     [rbp+150h+var_1C], 7 ; switch 8 cases
                ja      short def_7FF76B482238 ; jumptable 00007FF76B482238 default case
                movsxd  rax, [rbp+150h+var_1C]
                lea     rcx, cs:7FF76B470000h
                mov     eax, ds:(jpt_7FF76B482238 - 7FF76B470000h)[rcx+rax*4]
                add     rax, rcx
                jmp     rax             ; switch jump
loc_7FF76B48223A:       ; jumptable 00007FF76B482238 case 0
                mov     eax, [rbp+150h+var_10C]
                add     eax, 274h
                mov     [rbp+150h+var_10C], eax
loc_7FF76B482245:       ; jumptable 00007FF76B482238 case 1
                mov     eax, [rbp+150h+var_10C]
                sub     eax, 24Ch
                mov     [rbp+150h+var_10C], eax
loc_7FF76B482250:       ; jumptable 00007FF76B482238 case 2
                mov     eax, [rbp+150h+var_10C]
                add     eax, 2B0h
                mov     [rbp+150h+var_10C], eax
loc_7FF76B48225B:       ; jumptable 00007FF76B482238 case 3
                mov     eax, [rbp+150h+var_10C]
                sub     eax, 7Eh ; '~'
                mov     [rbp+150h+var_10C], eax
loc_7FF76B482264:       ; jumptable 00007FF76B482238 case 4
                mov     eax, [rbp+150h+var_10C]
                add     eax, 7Eh ; '~'
                mov     [rbp+150h+var_10C], eax
loc_7FF76B48226D:       ; jumptable 00007FF76B482238 case 5
                mov     eax, [rbp+150h+var_10C]
                sub     eax, 7Eh ; '~'
                mov     [rbp+150h+var_10C], eax
loc_7FF76B482276:       ; jumptable 00007FF76B482238 case 6
                mov     eax, [rbp+150h+var_10C]
                add     eax, 7Eh ; '~'
                mov     [rbp+150h+var_10C], eax
loc_7FF76B48227F:       ; jumptable 00007FF76B482238 case 7
                mov     eax, [rbp+150h+var_10C]
                sub     eax, 7Eh ; '~'
                mov     [rbp+150h+var_10C], eax
                jmp     short loc_7FF76B48228F
def_7FF76B482238:       ; jumptable 00007FF76B482238 default case
                call    j_explode_bomb
loc_7FF76B48228F:
                cmp     [rbp+150h+var_14C], 5
                jg      short loc_7FF76B48229D
                mov     eax, [rbp+150h+var_12C]
                cmp     [rbp+150h+var_10C], eax
                jz      short loc_7FF76B4822A2      ;End phase_3
loc_7FF76B48229D:
                call    j_explode_bomb
```
* `phase_3 disassembly`

We could see it took two input as integers as the format passed `lea rdx, aDD ; "%d %d"` before calling the `sscanf` function and checking if the number returns is `equal to 2 or not`.
```
mov     [rbp+150h+var_10C], 0
mov     [rbp+150h+var_EC], 0
lea     r9, [rbp+150h+var_12C]      ;second input
lea     r8, [rbp+150h+var_14C]      ;firstinput
lea     rdx, aDD        ; "%d %d"
mov     rcx, [rbp+150h+Buffer] ; Buffer
call    j_sscanf
mov     [rbp+150h+var_EC], eax
cmp     [rbp+150h+var_EC], 2
jge     short loc_7FF76B48220E
```
* `Input two integers`

And the first number should be less than 7 `(0:7)`. And we have a switch block depends on the first number we enter. So, if the first number in our input is less or equal to seven it moves an address depends on that number to the `RAX`. And depends on that address it jumps to a specific case inside the switch block doing some math`(add/sub)` operations till the end of switch block.
```
loc_7FF76B48220E:                       ; CODE XREF: phase_3+77↑j
                mov     eax, [rbp+150h+var_14C]         ;first input
                mov     [rbp+150h+var_1C], eax
                cmp     [rbp+150h+var_1C], 7 ; switch 8 cases
                ja      short def_7FF76B482238 ; jumptable 00007FF76B482238 default case
                movsxd  rax, [rbp+150h+var_1C]      ;rax = 0;
                lea     rcx, cs:7FF76B470000h
                mov     eax, ds:(jpt_7FF76B482238 - 7FF76B470000h)[rcx+rax*4]
                add     rax, rcx
                jmp     rax             ; switch jump
```
* `which address to jump to`

If we start from 0 (first case in switch) it `adds 274h` (628) then `sub 24Ch` (588) then `add 2B0h` (640) then `sub 7Eh` (126) then `add` it again then `sub` it and `add` it and `sub` it at the end of the switch case. After exiting the switch block it comperes the result with the second number we entered. So, after the switch cases the result for each number as a start for the switch block goes like `(0 --> 602 / 1 --> -26 / 2 --> 562 / 3 --> -126 / 4 --> 0 / 5 -->-126 / 6 --> 0 / 7 --> -126)`.
```
loc_7FF76B48223A:       ; jumptable 00007FF76B482238 case 0
                mov     eax, [rbp+150h+var_10C]
                add     eax, 274h
                mov     [rbp+150h+var_10C], eax
loc_7FF76B482245:       ; jumptable 00007FF76B482238 case 1
                mov     eax, [rbp+150h+var_10C]
                sub     eax, 24Ch
                mov     [rbp+150h+var_10C], eax
loc_7FF76B482250:       ; jumptable 00007FF76B482238 case 2
                mov     eax, [rbp+150h+var_10C]
                add     eax, 2B0h
                mov     [rbp+150h+var_10C], eax
loc_7FF76B48225B:       ; jumptable 00007FF76B482238 case 3
                mov     eax, [rbp+150h+var_10C]
                sub     eax, 7Eh ; '~'
                mov     [rbp+150h+var_10C], eax
loc_7FF76B482264:       ; jumptable 00007FF76B482238 case 4
                mov     eax, [rbp+150h+var_10C]
                add     eax, 7Eh ; '~'
                mov     [rbp+150h+var_10C], eax
loc_7FF76B48226D:       ; jumptable 00007FF76B482238 case 5
                mov     eax, [rbp+150h+var_10C]
                sub     eax, 7Eh ; '~'
                mov     [rbp+150h+var_10C], eax
loc_7FF76B482276:       ; jumptable 00007FF76B482238 case 6
                mov     eax, [rbp+150h+var_10C]
                add     eax, 7Eh ; '~'
                mov     [rbp+150h+var_10C], eax
loc_7FF76B48227F:       ; jumptable 00007FF76B482238 case 7
                mov     eax, [rbp+150h+var_10C]
                sub     eax, 7Eh ; '~'
                mov     [rbp+150h+var_10C], eax
                jmp     short loc_7FF76B48228F
def_7FF76B482238:       ; jumptable 00007FF76B482238 default case
                call    j_explode_bomb
```
* `Switch block`

But if we dig into the code, we could see it returns and compares the first input `if it's greater than 5` and if greater explode the bomb.
```
loc_7FF76B48228F:
                cmp     [rbp+150h+var_14C], 5
                jg      short loc_7FF76B48229D
                mov     eax, [rbp+150h+var_12C]
                cmp     [rbp+150h+var_10C], eax
                jz      short loc_7FF76B4822A2      ;End phase_3
loc_7FF76B48229D:
                call    j_explode_bomb
```
* `If greater than 5`

So, the answer for this phase after exclude both 6 aand 7 should be one of these two-number sets (`0 602`  `1 -26`  `2 232`  `3 -126`  `4 0`  `5 -126`)

And like you thought it works again😊
![result_Phase_3](/Bomb%20Lab/Phase_3%20defused.png)
* `phase_3 defused`


Yes, we are halfway of defusing the bomb congrats for us ^^_^^.


##Phase 4:
 
 From here things get more dirty and difficult. So, moving the breakpoint and disassemble `phase_4`

```
phase_4 proc near

                lea     r9, [rbp+170h+var_14C] ; second input
                lea     r8, [rbp+170h+var_16C] ; first input
                lea     rdx, aDD        ; "%d %d"
                mov     rcx, [rbp+170h+Buffer] ; Buffer
                call    j_sscanf
                mov     [rbp+170h+var_EC], eax
                cmp     [rbp+170h+var_EC], 2
                jnz     short loc_7FF7B3B823CD
                cmp     [rbp+170h+var_16C], 0
                jl      short loc_7FF7B3B823CD
                cmp     [rbp+170h+var_16C], 0Eh
                jle     short loc_7FF7B3B823D2
loc_7FF7B3B823CD:
                call    j_explode_bomb
loc_7FF7B3B823D2:
                mov     [rbp+170h+var_10C], 0Ah
                mov     r8d, 0Eh
                xor     edx, edx
                mov     ecx, [rbp+170h+var_16C]
                call    j_func4
                mov     [rbp+170h+var_12C], eax
                mov     eax, [rbp+170h+var_10C]
                cmp     [rbp+170h+var_12C], eax
                jnz     short loc_7FF7B3B823FC
                mov     eax, [rbp+170h+var_10C]
                cmp     [rbp+170h+var_14C], eax
                jz      short loc_7FF7B3B82401
loc_7FF7B3B823FC:
                call    j_explode_bomb
```
* `Phase_4 disassembly`

So, from here we could see it took two input as integers as the format passed `lea     rdx, aDD        ; "%d %d"` before calling the `sscanf` function and checking if the number returns is equal to 2 or not.
```
call    j_sscanf
mov     [rbp+170h+var_EC], eax
cmp     [rbp+170h+var_EC], 2
``` 
* `Inputs is 2`

After that it checks if the first number in input(saved in store `rbp+170h+var_16C`) is greater than `0`and below `14` which tells us our first input should be a number between these two numbers, now we have a little combination to try from😂.
```
cmp     [rbp+170h+var_16C], 0
jl      short loc_7FF7B3B823CD
cmp     [rbp+170h+var_16C], 0Eh
jle     short loc_7FF7B3B823D2
```
* `Is it in range?`

After making sure that the first input is in the range ` 0-14`, it starts passing some values to the registers used in another called function `func4`, and moving 10 to a `rbp+170h+var_10C` a store is being used later in a comparison with return value from `func4` and again with the second input we entered stored in `rbp+170h+var_14C` and that gives us a hint that the return value from that `func4` must be 10 and the seocnd input for `phase_4` is also 10. 
```
mov     [rbp+170h+var_12C], eax
mov     eax, [rbp+170h+var_10C]
cmp     [rbp+170h+var_12C], eax
jnz     short loc_7FF7B3B823FC
mov     eax, [rbp+170h+var_10C]
cmp     [rbp+170h+var_14C], eax
jz      short loc_7FF7B3B82401      ;end_phase_4
```
* `comparing with 10`

let's disassemble `func4` and see what it does...
```
                sub     eax, edx
                sar     eax, 1
                mov     ecx, [rbp+0F0h+arg_8]
                add     ecx, eax
                mov     eax, ecx
                mov     [rbp+0F0h+var_EC], eax
                mov     eax, [rbp+0F0h+arg_0]
                cmp     [rbp+0F0h+var_EC], eax
                jle     short loc_7FF7B3B81F9A
                mov     eax, [rbp+0F0h+var_EC]
                dec     eax
                mov     r8d, eax
                mov     edx, [rbp+0F0h+arg_8]
                mov     ecx, [rbp+0F0h+arg_0]
                call    j_func4
                add     eax, [rbp+0F0h+var_EC]
                jmp     short loc_7FF7B3B81FC8      ;end_func4
loc_7FF7B3B81F9A:                       ; CODE XREF: func4+68↑j
                mov     eax, [rbp+0F0h+arg_0]
                cmp     [rbp+0F0h+var_EC], eax
                jge     short loc_7FF7B3B81FC5
                mov     eax, [rbp+0F0h+var_EC]
                inc     eax
                mov     r8d, [rbp+0F0h+arg_10]
                mov     edx, eax
                mov     ecx, [rbp+0F0h+arg_0]
                call    j_func4
                add     eax, [rbp+0F0h+var_EC]
                jmp     short loc_7FF7B3B81FC8      ;end_func4
loc_7FF7B3B81FC5:                       ; CODE XREF: func4+93↑j
                mov     eax, [rbp+0F0h+var_EC]
                jmp     short loc_7FF7B3B81FC8      ;end_func4
```
* `func4 disassembly`

Inside `func4` we see a pattern of recursion for the function and as we saw before it must return with `10` and it actually accepts numbers between `0 14` many choices to try, better than trying to figure out what this function actually does, I tried brute force this function to see which number is actually returns with `10` and it was `3`.
So, let us assume that the answer for this phase is `3 10`.

And yeah, it works again ^^_^^:
![Phase_4 Defused](/Bomb%20Lab/Phase_4%20defused.png)
* `phase_4 defused`

##Phase 5:
Like usual put the breakpoint here and contiune through the Bomb. Disassemble `phase_5` gives us a note of what kind of inputs it's waiting for. So, let's see it...

```
                lea     r9, [rbp+190h+var_10C]      ;second input
                lea     r8, [rbp+190h+var_12C]      ;first input
                lea     rdx, aDD        ; "%d %d"
                mov     rcx, [rbp+190h+Buffer] ; Buffer
                call    j_sscanf
                mov     [rbp+190h+var_EC], eax
                cmp     [rbp+190h+var_EC], 2
                jge     short loc_7FF61E1324D9
                call    j_explode_bomb
loc_7FF61E1324D9:
                mov     eax, [rbp+190h+var_12C]
                and     eax, 0Fh
                mov     [rbp+190h+var_12C], eax
                mov     eax, [rbp+190h+var_12C]
                mov     [rbp+190h+var_14C], eax
                mov     [rbp+190h+var_18C], 0
                mov     [rbp+190h+var_16C], 0
loc_7FF61E1324F6:
                cmp     [rbp+190h+var_12C], 0Fh
                jz      short loc_7FF61E132524
                mov     eax, [rbp+190h+var_18C]
                inc     eax
                mov     [rbp+190h+var_18C], eax
                movsxd  rax, [rbp+190h+var_12C]
                lea     rcx, unk_7FF61E13F1D0
                mov     eax, [rcx+rax*4]
                mov     [rbp+190h+var_12C], eax
                mov     eax, [rbp+190h+var_12C]
                mov     ecx, [rbp+190h+var_16C]
                add     ecx, eax
                mov     eax, ecx
                mov     [rbp+190h+var_16C], eax
                jmp     short loc_7FF61E1324F6
loc_7FF61E132524:
                cmp     [rbp+190h+var_18C], 0Fh
                jnz     short loc_7FF61E132535
                mov     eax, [rbp+190h+var_10C]
                cmp     [rbp+190h+var_16C], eax
                jz      short loc_7FF61E13253A      ;end_phase_5
loc_7FF61E132535:
                call    j_explode_bomb
```
* `Phase_5 disassemble`

And we can notice what kinda of input it expects. Yes, like the previous two phases it took two integers
```
lea     r9, [rbp+190h+var_10C]      ;second input
lea     r8, [rbp+190h+var_12C]      ;first input
lea     rdx, aDD        ; "%d %d"
mov     rcx, [rbp+190h+Buffer] ; Buffer
call    j_sscanf
mov     [rbp+190h+var_EC], eax
cmp     [rbp+190h+var_EC], 2
jge     short loc_7FF61E1324D9
call    j_explode_bomb
```
* `Inputs is 2`

Then the action begins...
At first it does a `logical and (.)` for the firsrt input sotred in `rbp+190h+var_12C` with `F` which represnts all ones in four bits `1111` so it likes doing `and` with for ones, and like we know the result of anding with one is the same input `x . 1 = x`, the result from this section is actually the same input then moves `zero` to some stores to use them in a coming loop `rbp+190h+var_18C` as a counter and `rbp+190h+var_16C` as a store for add operation.
```
loc_7FF61E1324D9:
                mov     eax, [rbp+190h+var_12C]
                and     eax, 0Fh
                mov     [rbp+190h+var_12C], eax
                mov     eax, [rbp+190h+var_12C]
                mov     [rbp+190h+var_14C], eax
                mov     [rbp+190h+var_18C], 0
                mov     [rbp+190h+var_16C], 0
```
* `Anding and zeros`

The next block it turns out to be a loop block. It starts with comparing our first input with `15(Fh)` and if below continue excuting the loop.  
```
loc_7FF61E1324F6:
                cmp     [rbp+190h+var_12C], 0Fh
                jz      short loc_7FF61E132524
                mov     eax, [rbp+190h+var_18C]
                inc     eax
                mov     [rbp+190h+var_18C], eax
                movsxd  rax, [rbp+190h+var_12C]
                lea     rcx, unk_7FF61E13F1D0
                mov     eax, [rcx+rax*4]
                mov     [rbp+190h+var_12C], eax
                mov     eax, [rbp+190h+var_12C]
                mov     ecx, [rbp+190h+var_16C]
                add     ecx, eax
                mov     eax, ecx
                mov     [rbp+190h+var_16C], eax
                jmp     short loc_7FF61E1324F6
```
* `Loop Block`

```
cmp     [rbp+190h+var_12C], 0Fh
jz      short loc_7FF61E132524
```
* `If we reach 15`

After that some sort of counter to see how many iterations this loop takes.
And, it passes the value of the first input to `RAX` and load the address `7FF61E1324D9` to `RCX` and if we look at that address we'll see it has the value `10(Ah)`, then passes a value stored in address that appears to be an index of an array to `eax`.
```
mov     eax, [rbp+190h+var_18C]
inc     eax
mov     [rbp+190h+var_18C], eax
```
* `The counter`

```
movsxd  rax, [rbp+190h+var_12C]
lea     rcx, unk_7FF61E13F1D0
mov     eax, [rcx+rax*4]
mov     [rbp+190h+var_12C], eax
```
* `Index process and return to eax`

And, if we continue through the memory and do the math for this index address it appears for us we are dealing with an `array` with `16` elements `from 0 to 15` arranged in some order, and every element of this array works as a pointer in this operation `rcx+rax*4` to another element inside the array. Our goal now is to reach `15` to get out of the loop after `15` iterations. After a quick analysis and reverse the process, I found the start number shall be `5` to reach `15` at the end the goal here and our exiting condition from the loop after `15` iterations. 

![phase_5_memory](/Bomb%20Lab/phase_5_pattern.png)
* `Elemnts in the array`
`Using the start address stored in RCX (7FF61E13F1D0) and the index operation rcx+rax*4 you can follow the numbers in the previous image.`

But keep in your mind that after every return process it adds the return value from our array to `rbp+190h+var_16C` without the first number we start with which is gonna be the sum of the numbers from `0` to `15` excepts `5` and it's `115` and returns to comapre this result with our second input. At the end, this section must return with three values: `15` in `rbp+190h+var_12C` and also in the counter `rbp+190h+var_18C` and the sum of this array `excepts 5` in `rbp+190h+var_16C`
```
mov     eax, [rbp+190h+var_12C]
mov     ecx, [rbp+190h+var_16C]
add     ecx, eax
mov     eax, ecx
mov     [rbp+190h+var_16C], eax
jmp     short loc_7FF61E1324F6
```
* `Add every return elements from the array`


```
loc_7FF61E132524:
                cmp     [rbp+190h+var_18C], 0Fh
                jnz     short loc_7FF61E132535
                mov     eax, [rbp+190h+var_10C]
                cmp     [rbp+190h+var_16C], eax
                jz      short loc_7FF61E13253A      ;end_phase_5
loc_7FF61E132535:
                call    j_explode_bomb
```
* `Comaring the results from the loop`

So, now we have a good idea of what should our inputs be. 
Yes, it's `5 115`

Great work for us really we got here😊.
![Phase_5 defused](/Bomb%20Lab/Phase_5%20defused.png)
* `phase_5 defused`

##Phase 6:

Now to the final phase🫣 in our bomb `phase_6`.  It looks like a bomb really but as usual we're gonna destroy it. At first look at the assembly you might freak out but don't worry we'll break it down into small pieces we can actually deal with...

```
                call    j_read_six_numbers
                mov     [rbp+1D0h+var_10C], 0
                jmp     short loc_7FF7EADD262C
loc_7FF7EADD261E:
                mov     eax, [rbp+1D0h+var_10C]
                inc     eax
                mov     [rbp+1D0h+var_10C], eax
loc_7FF7EADD262C:
                cmp     [rbp+1D0h+var_10C], 6
                jge     short loc_7FF7EADD269E
                movsxd  rax, [rbp+1D0h+var_10C]
                cmp     [rbp+rax*4+1D0h+var_188], 1
                jl      short loc_7FF7EADD2651
                movsxd  rax, [rbp+1D0h+var_10C]
                cmp     [rbp+rax*4+1D0h+var_188], 6
                jle     short loc_7FF7EADD2656
loc_7FF7EADD2651:
                call    j_explode_bomb
loc_7FF7EADD2656:
                mov     eax, [rbp+1D0h+var_10C]
                inc     eax
                mov     [rbp+1D0h+var_EC], eax
                jmp     short loc_7FF7EADD2674
loc_7FF7EADD2666:
                mov     eax, [rbp+1D0h+var_EC]
                inc     eax
                mov     [rbp+1D0h+var_EC], eax
loc_7FF7EADD2674:
                cmp     [rbp+1D0h+var_EC], 6
                jge     short loc_7FF7EADD269C
loc_7FF7EADD269A:
                jmp     short loc_7FF7EADD2666
loc_7FF7EADD269C:
                jmp     short loc_7FF7EADD261E
loc_7FF7EADD269E:
                mov     [rbp+1D0h+var_10C], 0
                jmp     short loc_7FF7EADD26B8
loc_7FF7EADD26AA:
                mov     eax, [rbp+1D0h+var_10C]
                inc     eax
                mov     [rbp+1D0h+var_10C], eax
loc_7FF7EADD26B8:
                cmp     [rbp+1D0h+var_10C], 6
                jge     short loc_7FF7EADD2716
loc_7FF7EADD26D5:
                mov     eax, [rbp+1D0h+var_EC]
                inc     eax
                mov     [rbp+1D0h+var_EC], eax
loc_7FF7EADD26E3:
                movsxd  rax, [rbp+1D0h+var_10C]
                mov     eax, [rbp+rax*4+1D0h+var_188]
                cmp     [rbp+1D0h+var_EC], eax
                jge     short loc_7FF7EADD2704
loc_7FF7EADD2704:
                movsxd  rax, [rbp+1D0h+var_10C]
                mov     rcx, [rbp+1D0h+var_1A8]
                mov     [rbp+rax*8+1D0h+var_158], rcx
                jmp     short loc_7FF7EADD26AA
loc_7FF7EADD2716:
                mov     eax, 8
                imul    rax, 0
                mov     rax, [rbp+rax+1D0h+var_158]
                mov     [rbp+1D0h+var_1C8], rax
                mov     rax, [rbp+1D0h+var_1C8]
                mov     [rbp+1D0h+var_1A8], rax
                mov     [rbp+1D0h+var_10C], 1
                jmp     short loc_7FF7EADD274A
loc_7FF7EADD273C:
                mov     eax, [rbp+1D0h+var_10C]
                inc     eax
                mov     [rbp+1D0h+var_10C], eax
loc_7FF7EADD274A:
                cmp     [rbp+1D0h+var_10C], 6
                jge     short loc_7FF7EADD2775
loc_7FF7EADD2775:
                mov     rax, [rbp+1D0h+var_1A8]
                mov     qword ptr [rax+8], 0
                mov     rax, [rbp+1D0h+var_1C8]
                mov     [rbp+1D0h+var_1A8], rax
                mov     [rbp+1D0h+var_10C], 0
                jmp     short loc_7FF7EADD27A3
loc_7FF7EADD2795:
                mov     eax, [rbp+1D0h+var_10C]
                inc     eax
                mov     [rbp+1D0h+var_10C], eax
loc_7FF7EADD27A3:
                cmp     [rbp+1D0h+var_10C], 5
                jge     short loc_7FF7EADD27D1      ;end_phase_6
                mov     rax, [rbp+1D0h+var_1A8]
                mov     rax, [rax+8]
                mov     rcx, [rbp+1D0h+var_1A8]
                mov     eax, [rax]
                cmp     [rcx], eax
                jge     short loc_7FF7EADD27C3
loc_7FF7EADD27C3:
                mov     rax, [rbp+1D0h+var_1A8]
                mov     rax, [rax+8]
                mov     [rbp+1D0h+var_1A8], rax
                jmp     short loc_7FF7EADD2795
```
* `pase_6 disassembly`

Like `phase_2` this phase is actually dealing with six numbers. We could obviously see it calls a function called `read_six_numbers`, and if we disassemble it we would found that it reads six integers as input with function `sscanf`.
```
            lea     rdx, aDDDDDD    ; "%d %d %d %d %d %d"
            mov     rcx, [rbp+0F0h+Buffer] ; Buffer
            call    j_sscanf
            mov     [rbp+0F0h+var_EC], eax
            cmp     [rbp+0F0h+var_EC], 6
            jge     short loc_7FF76B48306E
            call    j_explode_bomb
```
* `read_six_numbers disassembly`

we can see it loads some format into `rdx` "%d" which is the format for integer number then it calls `sscanf` function which returns with the number of inputs we entered and sotre it in `eax` then compare it to 6
```
lea     rdx, aDDDDDD    ; "%d %d %d %d %d %d"       ;integer format
mov     rcx, [rbp+0F0h+Buffer] ; Buffer
call    j_sscanf
mov     [rbp+0F0h+var_EC], eax
cmp     [rbp+0F0h+var_EC], 6
jge     short loc_7FF76B48306E
```
* `input 6 numbers`

After taking six inputs and sotre it into some kind of storage it checks at first if the numbers we entered is `between 1 and 6` and if one is not explode the bomb. While doing this check it also makes sure that the six inputs are different and there is no repeat. That gives us a clue that our input is the set of number from 1 to 6 but in what order?

```
loc_7FF7EADD261E:
                mov     eax, [rbp+1D0h+var_10C]
                inc     eax
                mov     [rbp+1D0h+var_10C], eax
loc_7FF7EADD262C:
                cmp     [rbp+1D0h+var_10C], 6
                jge     short loc_7FF7EADD269E
                movsxd  rax, [rbp+1D0h+var_10C]
                cmp     [rbp+rax*4+1D0h+var_188], 1
                jl      short loc_7FF7EADD2651
                movsxd  rax, [rbp+1D0h+var_10C]
                cmp     [rbp+rax*4+1D0h+var_188], 6
                jle     short loc_7FF7EADD2656
```
* `The number is in range`

This huge block of code is really doing a little check that all the number we entered are different and the is no repeat. So we got a pointer that refer to the element being checked `rbp+rax*4+1D0h+var_188` and a pointer that move through other inpunts `rbp+rcx*4+1D0h+var_188`.
```
loc_7FF7EADD2656:
                mov     eax, [rbp+1D0h+var_10C]
                inc     eax
                mov     [rbp+1D0h+var_EC], eax
                jmp     short loc_7FF7EADD2674
loc_7FF7EADD2666:
                mov     eax, [rbp+1D0h+var_EC]
                inc     eax
                mov     [rbp+1D0h+var_EC], eax
loc_7FF7EADD2674:
                cmp     [rbp+1D0h+var_EC], 6
                jge     short loc_7FF7EADD269C
                movsxd  rax, [rbp+1D0h+var_10C]
                movsxd  rcx, [rbp+1D0h+var_EC]
                mov     ecx, [rbp+rcx*4+1D0h+var_188]
                cmp     [rbp+rax*4+1D0h+var_188], ecx
                jnz     short loc_7FF7EADD269A
                call    j_explode_bomb
loc_7FF7EADD269A:
                jmp     short loc_7FF7EADD2666
loc_7FF7EADD269C:
                jmp     short loc_7FF7EADD261E      ;go back to the counter and start with another number in our list
```
* `No repeat here`

The next section in the phase it seems doing re-arrange the input in some order. But, when we look at the memory and look at the changes of the registers we notice we're dealing with `Linked List` has `six nodes` and every input of ours refers to a node in this linked list. So this section arrange the nodes of the linked list based on the order we entered our inputs. If we entered `1 2 3 4 5 6 ` the nodes must be order like `node1 node2 node3 node4 node5 node6`. And ofcourse these nodes has data in it which is gonna be our hero in the next section of the phase.
```
loc_7FF7EADD269E:
                mov     [rbp+1D0h+var_10C], 0
                jmp     short loc_7FF7EADD26B8
loc_7FF7EADD26AA:
                mov     eax, [rbp+1D0h+var_10C]
                inc     eax
                mov     [rbp+1D0h+var_10C], eax
loc_7FF7EADD26B8:
                cmp     [rbp+1D0h+var_10C], 6
                jge     short loc_7FF7EADD2716
loc_7FF7EADD26D5:
                mov     eax, [rbp+1D0h+var_EC]
                inc     eax
                mov     [rbp+1D0h+var_EC], eax
loc_7FF7EADD26E3:
                movsxd  rax, [rbp+1D0h+var_10C]
                mov     eax, [rbp+rax*4+1D0h+var_188]
                cmp     [rbp+1D0h+var_EC], eax
                jge     short loc_7FF7EADD2704
loc_7FF7EADD2704:
                movsxd  rax, [rbp+1D0h+var_10C]
                mov     rcx, [rbp+1D0h+var_1A8]
                mov     [rbp+rax*8+1D0h+var_158], rcx
                jmp     short loc_7FF7EADD26AA
loc_7FF7EADD2716:
                mov     eax, 8
                imul    rax, 0
                mov     rax, [rbp+rax+1D0h+var_158]
                mov     [rbp+1D0h+var_1C8], rax
                mov     rax, [rbp+1D0h+var_1C8]
                mov     [rbp+1D0h+var_1A8], rax
                mov     [rbp+1D0h+var_10C], 1
                jmp     short loc_7FF7EADD274A
loc_7FF7EADD273C:
                mov     eax, [rbp+1D0h+var_10C]
                inc     eax
                mov     [rbp+1D0h+var_10C], eax
loc_7FF7EADD274A:
                cmp     [rbp+1D0h+var_10C], 6
                jge     short loc_7FF7EADD2775
```

After re-arrange the linked list to catch up our input list, it starts again doing check for the nodes but this time for the data stored in it...
The right order for our input, so be the right order for the linked list nodes must achieve a condition which is the data stored in each node must be greater than the next to it. 
```
loc_7FF7EADD2775:
                mov     rax, [rbp+1D0h+var_1A8]
                mov     qword ptr [rax+8], 0
                mov     rax, [rbp+1D0h+var_1C8]
                mov     [rbp+1D0h+var_1A8], rax
                mov     [rbp+1D0h+var_10C], 0
                jmp     short loc_7FF7EADD27A3
loc_7FF7EADD2795:
                mov     eax, [rbp+1D0h+var_10C]
                inc     eax
                mov     [rbp+1D0h+var_10C], eax
loc_7FF7EADD27A3:
                cmp     [rbp+1D0h+var_10C], 5
                jge     short loc_7FF7EADD27D1      ;end_phase_6
                mov     rax, [rbp+1D0h+var_1A8]
                mov     rax, [rax+8]
                mov     rcx, [rbp+1D0h+var_1A8]
                mov     eax, [rax]
                cmp     [rcx], eax
                jge     short loc_7FF7EADD27C3
loc_7FF7EADD27C3:
                mov     rax, [rbp+1D0h+var_1A8]
                mov     rax, [rax+8]
                mov     [rbp+1D0h+var_1A8], rax
                jmp     short loc_7FF7EADD2795
```
* `Check if they arranged`

So, we must know the data stored inside each node and arrange them....
![Phase_6 memory](/Bomb%20Lab/phase_6_nodes.png)
* `Phase 6 memory`

After looking in the memory we could figure out the right arrange that achieve our condition is `5 4 3 1 6 2`

Finally we defused the Bomb, Yeahhhh💪💪
![Phase 6 Defused](/Bomb%20Lab/Phase_6%20defused.png)
* `phase_6 defused`


###Secret Phase:

This phase has two condition to be activated one is really obvious if we disassemble the function `phase_defused` and it's that we must finish all the six phases which we did. The other condition is in the input we enter, but it doesn't ask for input after `phase_6` defused which means it's inputs are from previous phase and if we look at the inputs it takes to be activated, they are two integers and a string...
So, we must enter a string into a phase that took two integers as input and we have 3 options `phase_3`, `phase_4`, `phase_5`

```
                cmp     cs:num_input_strings, 6
                jnz     loc_7FF7D3D52D4D
                mov     eax, 50h ; 'P'
                imul    rax, 3
                lea     rcx, input_strings
                add     rcx, rax
                mov     rax, rcx
                lea     rcx, [rbp+1A0h+var_190]
                mov     [rsp+1D0h+var_1B0], rcx
                lea     r9, [rbp+1A0h+var_10C]
                lea     r8, [rbp+1A0h+var_12C]
                lea     rdx, aDDS       ; "%d %d %s"
                mov     rcx, rax        ; Buffer
                call    j_sscanf
                mov     [rbp+1A0h+var_EC], eax
                cmp     [rbp+1A0h+var_EC], 3
                jnz     short loc_7FF7D3D52D41
                lea     rdx, aDrevil    ; "DrEvil"
                lea     rcx, [rbp+1A0h+var_190]
                call    j_strings_not_equal
                test    eax, eax
                jnz     short loc_7FF7D3D52D41
                lea     rcx, aCursesYouVeFou ; "Curses, you've found the secret phase!"...
                call    j_printf
                lea     rcx, aButFindingItAn ; "But finding it and solving it are quite"...
                call    j_printf
                call    j_secret_phase
loc_7FF7D3D52D41:
                lea     rcx, aCongratulation ; "Congratulations! You've defused the bom"...
                call    j_printf
```
* `three inputs: two integers and a string`

To figure out which phase is the one we're looking for, we are gonna check the pramaters this function takes `RCX/RAX` and see what does it stores...
![RCX memory](/Bomb%20Lab/RCX.png)
* `RAX/RCX in the memory`

From the memory we could see `RCX` points to `3 10` which is the input for `phase_4`, but it has a missing input here which is the string the third one for activation.
If we continue through the code we see it loading some string to `RDX` and took the string assuming we entered as parameters for function `string_not_equal` and that string is `DrEvil`.
So go back and try again with the Bomb and enter `3 10 DrEvil` as inputs for `phase_4`

```
lea     rdx, aDrevil    ; "DrEvil"
lea     rcx, [rbp+1A0h+var_190]
call    j_strings_not_equal
test    eax, eax
jnz     short loc_7FF7EADD2D41
ea     rcx, aCursesYouVeFou ; "Curses, you've found the secret phase!"...
call    j_printf
lea     rcx, aButFindingItAn ; "But finding it and solving it are quite"...
call    j_printf
call    j_secret_phase      ;the mystry phase we're looking for
loc_7FF7EADD2D41:
lea     rcx, aCongratulation ; "Congratulations! You've defused the bom"...
call    j_printf
```
* `phase_defused disassembly`

![Secret phase shows up](/Bomb%20Lab/Secret%20phase%20shows%20up.png)
* `Secret phase shows up`

And now we can enter the `secret_phase`...
If we dig inside this phase, we'll see it calling function `func7`. And the input for this phase is an integer less than `1001` as we can see it calls `imp_atoi` that converts the string into integers. 

```
                call    j_read_line
                mov     [rbp+130h+String], rax
                mov     rcx, [rbp+130h+String] ; String
                call    cs:__imp_atoi
                mov     [rbp+130h+var_10C], eax
                cmp     [rbp+130h+var_10C], 1
                jl      short loc_7FF7EADD28DF
                cmp     [rbp+130h+var_10C], 3E9h
                jle     short loc_7FF7EADD28E4
loc_7FF7EADD28DF:
                call    j_explode_bomb
loc_7FF7EADD28E4:
                mov     edx, [rbp+130h+var_10C]
                lea     rcx, n1
                call    j_fun7
                mov     [rbp+130h+var_EC], eax
                cmp     [rbp+130h+var_EC], 5
                jz      short loc_7FF7EADD2901
                call    j_explode_bomb
```
* `secret_phase disassembly`
so let's look at `func7` to see what kind of operations it does and try to figure out the input it deals with.....

```
                cmp     [rbp+0D0h+arg_0], 0
                jnz     short loc_7FF7EADD1E7B
                mov     eax, 0FFFFFFFFh
                jmp     short loc_7FF7EADD1ED9      ;end secret_phase
loc_7FF7EADD1E7B:                       ; CODE XREF: fun7+42↑j
                mov     rax, [rbp+0D0h+arg_0]
                mov     eax, [rax]
                cmp     dword ptr [rbp+0D0h+arg_8], eax
                jge     short loc_7FF7EADD1EA8
                mov     edx, dword ptr [rbp+0D0h+arg_8]
                mov     rax, [rbp+0D0h+arg_0]
                mov     rcx, [rax+8]
                call    j_fun7
                shl     eax, 1
                jmp     short loc_7FF7EADD1ED9
loc_7FF7EADD1EA8:                       ; CODE XREF: fun7+5A↑j
                mov     rax, [rbp+0D0h+arg_0]
                mov     eax, [rax]
                cmp     dword ptr [rbp+0D0h+arg_8], eax
                jnz     short loc_7FF7EADD1EBF
                or     eax, eax
                jmp     short loc_7FF7EADD1ED9
loc_7FF7EADD1EBF:                       ; CODE XREF: fun7+87↑j
                mov     edx, dword ptr [rbp+0D0h+arg_8]
                mov     rax, [rbp+0D0h+arg_0]
                mov     rcx, [rax+10h]
                all    j_fun7
                lea     eax, [rax+rax+1]
```
* `func7 disassembly`

Inside `func7` we can see that it's a mix of `phase_4` and `phase_6` principles. It has the recursion principle and a linked list with `15 nodes`.
If we follow these nodes at the memory we can see the data it stores, and it turns out to be all integers, and it compares those data with our input for this phase, so the input for `secret_phase`shall be integer as well.
`(n1 --> 36) (n21 --> 8) (n22 --> 50) (n31 --> 6) (n32 --> 22) (n33 --> 45) (n34 --> 107) (n41 --> 1) (n42 --> 7) (n43 --> 20) (n44 --> 35) (n45 -->40) (n46 --> 47) (n47 --> 99) (n48 --> 1001)`

![Secret_phase memory](/Bomb%20Lab/secret_phase_nodes.png)
* `Secret Phase Memory`
```
loc_7FF7EADD1E7B:                       ; CODE XREF: fun7+42↑j
                mov     rax, [rbp+0D0h+arg_0]
                mov     eax, [rax]
                cmp     dword ptr [rbp+0D0h+arg_8], eax
                jge     short loc_7FF7EADD1EA8
                mov     edx, dword ptr [rbp+0D0h+arg_8]
                mov     rax, [rbp+0D0h+arg_0]
                mov     rcx, [rax+8]
                call    j_fun7
                shl     eax, 1
                jmp     short loc_7FF7EADD1ED9
loc_7FF7EADD1EA8:                       ; CODE XREF: fun7+5A↑j
                mov     rax, [rbp+0D0h+arg_0]
                mov     eax, [rax]
                cmp     dword ptr [rbp+0D0h+arg_8], eax
                jnz     short loc_7FF7EADD1EBF
                or      eax, eax
                jmp     short loc_7FF7EADD1ED9
loc_7FF7EADD1EBF:                       ; CODE XREF: fun7+87↑j
                mov     edx, dword ptr [rbp+0D0h+arg_8]
                mov     rax, [rbp+0D0h+arg_0]
                mov     rcx, [rax+10h]
                all    j_fun7
                lea     eax, [rax+rax+1]
```
* `func7 recursion`

We have many choices to try, so...
As we did with `phase_4`, we're gonna do here with `func7`. yes, brute force it untill this function return with `5` in `EAX` the flag defused this phase.
```
loc_7FF7EADD28E4:
                mov     edx, [rbp+130h+var_10C]
                lea     rcx, n1
                call    j_fun7
                mov     [rbp+130h+var_EC], eax
                cmp     [rbp+130h+var_EC], 5
                jz      short loc_7FF7EADD2901
                call    j_explode_bomb
```

So the value that returns with `5` is `47`

And yes, at the end we manage to defuse the Binary Bomb Lab😇😇
![Bomb final defused](/Bomb%20Lab/Bomb%20finally%20defused.png)
* `Binary Bomb Lab Defused`
