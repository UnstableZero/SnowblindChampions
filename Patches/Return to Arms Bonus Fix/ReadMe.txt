The patch files contained in this folder are intentionally formatted.
Modifying these codes to have the RAW information WILL cause issues with PS2_patch_engine.
I would not recommend trying this on a disc.

These files are formatted to work with PS2_patch_engine and/or PCSX2's patch system.
Use of PS2_patch_engine allows the creation of a permanent .iso where bonus fix patch is permanently applied.
Doing this will cause the CRC of the game to change, I would recommend creating new cheat and patch files for this copy of the game.
This patch is designed to work entirely within the constraints of the original executable for the game.
Not all cheats/mods will be compatible, as some addresses within RecomputeAbilityModifiers are now different.
For ease of use, the PNACH files have no comments, and are fine to copy paste directly into PS2_patch_engine.

If you're using PCSX2, please add this block to the top of the patch for a toggle button.

[HP/MP Bonus Fix]
author=Switz0018
description=Hacks in functionality of the health and mana bonus.


Some notes:

RTA calculates health and mana bonus, and the proper totals correctly, just at the wrong time.
In the Alpha version of the game, the function known as RecomputeAbilityModifiers calls the function RecomputePaperdollModifiers, and applies bonuses to health and mana correctly.
Figurines do not work in this version in the game.
Figurines work in the Release version, but health and mana bonus don't.
Thanks to Ghidra, when comparing the Alpha and Release, we see that RecomputePaperdollModifiers is called at two entirely different steps of RecomputeAbilityModifiers.
This is what makes figurines work, I checked. So what's the problem with health and mana?
They're saved to where your normal max health and mana should be saved, and then overwritten moments later.
There are several solutions to this problem, I tried a couple that I could imagine, but this is what ultimately produced a safe and usable result.
First step was to stop saving the health and mana bonus to where they get overwritten, easy, use a couple empty registers. Done.
We also have to stop loading your health and mana from character data, we don't care about that anymore.
Now we have bonus health and mana in a couple floating point registers.
Next, we need to add our bonuses to health and mana. This is the hard part.
In order to affect the game as little as possible, I need to stay within the same file size, and preferably, affect as little else of the game as possible.
Within roughly 150 instructions inside RecomputeAbilityModifiers there are four spots with double "nop", meaning, do nothing, twice.
Better yet, the first one is always after a branch, we never see the second one, it's wasted space.
So I manually did some hex edit to remove and add bytes where I needed them, and then made sure I did no damage by comparing with Ghidra.
Of course there's damage! But only a little, 10 branches, fixed.
Now I've moved 16 bytes(4 instructions), to just before we save calculated health and mana, based on starting math, and ability points.
These instructions are used to add hp and mana bonus to the correct registers, and then clearing the two registers we've made use of.
We must clear these registers immediately, as they're in a loop used to calculate all four players, and carrying values around is bad.
Finally, by applying this patch with PS2_patch_engine and checking with Ghidra, we can see the psuedo-C is almost entirely the same except for where we add two extra variables we made in RecomputePaperdollModifiers.
Below is the patch with some comments.

Thanks for reading. -Switz0018

//Fix references to fig/bonus check.
//patch=1,EE,001B67BC,byte,0000005F
//patch=1,EE,001B6808,byte,0000004C
//patch=1,EE,001B688C,byte,0000002B
//patch=1,EE,001B68C0,byte,0000001E
//Fix references to each class' starting math.
//patch=1,EE,001b6780,word,10006014
//patch=1,EE,001b67c4,word,12006214
//patch=1,EE,001b6810,word,0d006214
//patch=1,EE,001b6848,word,12006214
//patch=1,EE,001b6894,word,0c006214
//patch=1,EE,001b6840,word,1000001b
//The above codes are contained in the main body, but here for reference.

[HP/MP Bonus Fix]
author=Switz0018
description=Hacks in functionality of the health and mana bonus.
//Use fp30/31 for bonus HP and MP calculation
//Health
patch=1,EE,001b5c4c,word,461D0002
patch=1,EE,001b5c68,word,4600FFC0
//Mana
patch=1,EE,001b5cac,word,461D0002
patch=1,EE,001b5cb4,word,4600F780
//The patch to move 16 spare bytes from the middle of the function to a block where we use them to add bonus health and mana.
patch=1,EE,001b6780,word,14600010
patch=1,EE,001b6784,word,24020002
patch=1,EE,001b6788,word,3c014150
patch=1,EE,001b678c,word,4481b800
patch=1,EE,001b6790,word,3c013fc0
patch=1,EE,001b6794,word,4481c000
patch=1,EE,001b6798,word,3c0140c0
patch=1,EE,001b679c,word,4481c800
patch=1,EE,001b67a0,word,3c014168
patch=1,EE,001b67a4,word,4481b000
patch=1,EE,001b67a8,word,3c014019
patch=1,EE,001b67ac,word,34219999
patch=1,EE,001b67b0,word,4481a000
patch=1,EE,001b67b4,word,3c014282
patch=1,EE,001b67b8,word,4481a800
patch=1,EE,001b67bc,word,1000005f
patch=1,EE,001b67c0,word,00000000
patch=1,EE,001b67c4,word,14620012
patch=1,EE,001b67c8,word,24020001
patch=1,EE,001b67cc,word,3c0141c0
patch=1,EE,001b67d0,word,4481b800
patch=1,EE,001b67d4,word,3c01400c
patch=1,EE,001b67d8,word,3421cccc
patch=1,EE,001b67dc,word,4481c000
patch=1,EE,001b67e0,word,3c01409c
patch=1,EE,001b67e4,word,3421cccc
patch=1,EE,001b67e8,word,4481c800
patch=1,EE,001b67ec,word,3c014130
patch=1,EE,001b67f0,word,4481b000
patch=1,EE,001b67f4,word,3c014006
patch=1,EE,001b67f8,word,34216666
patch=1,EE,001b67fc,word,4481a000
patch=1,EE,001b6800,word,3c014224
patch=1,EE,001b6804,word,4481a800
patch=1,EE,001b6808,word,1000004c
patch=1,EE,001b680c,word,00000000
patch=1,EE,001b6810,word,1462000d
patch=1,EE,001b6814,word,24020004
patch=1,EE,001b6818,word,3c0141c0
patch=1,EE,001b681c,word,4481b800
patch=1,EE,001b6820,word,3c01400c
patch=1,EE,001b6824,word,3421cccc
patch=1,EE,001b6828,word,4481c000
patch=1,EE,001b682c,word,3c0140a3
patch=1,EE,001b6830,word,34213333
patch=1,EE,001b6834,word,4481c800
patch=1,EE,001b6838,word,3c014140
patch=1,EE,001b683c,word,4481b000
patch=1,EE,001b6840,word,1000001b
patch=1,EE,001b6844,word,00000000
patch=1,EE,001b6848,word,14620012
patch=1,EE,001b684c,word,24020003
patch=1,EE,001b6850,word,3c0141c0
patch=1,EE,001b6854,word,4481b800
patch=1,EE,001b6858,word,3c014019
patch=1,EE,001b685c,word,34219999
patch=1,EE,001b6860,word,4481c000
patch=1,EE,001b6864,word,3c0140d0
patch=1,EE,001b6868,word,4481c800
patch=1,EE,001b686c,word,3c014119
patch=1,EE,001b6870,word,34219999
patch=1,EE,001b6874,word,4481b000
patch=1,EE,001b6878,word,3c013fcc
patch=1,EE,001b687c,word,3421cccc
patch=1,EE,001b6880,word,4481a000
patch=1,EE,001b6884,word,3c0141b0
patch=1,EE,001b6888,word,4481a800
patch=1,EE,001b688c,word,1000002b
patch=1,EE,001b6890,word,00000000
patch=1,EE,001b6894,word,1462000c
patch=1,EE,001b6898,word,24020005
patch=1,EE,001b689c,word,3c014170
patch=1,EE,001b68a0,word,4481b800
patch=1,EE,001b68a4,word,3c014141
patch=1,EE,001b68a8,word,34219999
patch=1,EE,001b68ac,word,4481b000
patch=1,EE,001b68b0,word,3c014000
patch=1,EE,001b68b4,word,4481a000
patch=1,EE,001b68b8,word,3c0141f0
patch=1,EE,001b68bc,word,4481a800
patch=1,EE,001b68c0,word,1000001e
patch=1,EE,001b68c4,word,00000000
patch=1,EE,001b68c8,word,1462000d
patch=1,EE,001b68cc,word,24020006
patch=1,EE,001b68d0,word,3c014170
patch=1,EE,001b68d4,word,4481b800
patch=1,EE,001b68d8,word,3c014146
patch=1,EE,001b68dc,word,34216666
patch=1,EE,001b68e0,word,4481b000
patch=1,EE,001b68e4,word,3c014006
patch=1,EE,001b68e8,word,34216666
patch=1,EE,001b68ec,word,4481a000
patch=1,EE,001b68f0,word,3c014218
patch=1,EE,001b68f4,word,4481a800
patch=1,EE,001b68f8,word,10000010
patch=1,EE,001b68fc,word,00000000
patch=1,EE,001b6900,word,1462000e
patch=1,EE,001b6904,word,00000000
patch=1,EE,001b6908,word,3c0141c0
patch=1,EE,001b690c,word,4481b800
patch=1,EE,001b6910,word,3c014019
patch=1,EE,001b6914,word,34219999
patch=1,EE,001b6918,word,4481c000
patch=1,EE,001b691c,word,3c014129
patch=1,EE,001b6920,word,34219999
patch=1,EE,001b6924,word,4481b000
patch=1,EE,001b6928,word,3c013ff3
patch=1,EE,001b692c,word,34213333
patch=1,EE,001b6930,word,4481a000
patch=1,EE,001b6934,word,3c0141f8
patch=1,EE,001b6938,word,4481a800
patch=1,EE,001b693c,word,51600006
patch=1,EE,001b6940,word,82030059
patch=1,EE,001b6944,word,0c06d5e4
patch=1,EE,001b6948,word,0220202d
patch=1,EE,001b694c,word,8e0b43d8
patch=1,EE,001b6950,word,a56200e4
patch=1,EE,001b6954,word,82030059
patch=1,EE,001b6958,word,c604009c
patch=1,EE,001b695c,word,46802120
patch=1,EE,001b6960,word,c6000094
patch=1,EE,001b6964,word,46800020
patch=1,EE,001b6968,word,8e020090
patch=1,EE,001b696c,word,44831800
patch=1,EE,001b6970,word,468018e0
patch=1,EE,001b6974,word,8d6403c8
patch=1,EE,001b6978,word,2442007d
patch=1,EE,001b697c,word,24050040
patch=1,EE,001b6980,word,46142102
patch=1,EE,001b6984,word,46170001
patch=1,EE,001b6988,word,46161842
patch=1,EE,001b698c,word,461918c2
patch=1,EE,001b6990,word,44821000
patch=1,EE,001b6994,word,468010a0
patch=1,EE,001b6998,word,46180002
patch=1,EE,001b699c,word,46040840
patch=1,EE,001b69a0,word,e6020124
patch=1,EE,001b69a4,word,46030000
patch=1,EE,001b69a8,word,46150841
patch=1,EE,001b69ac,word,461f0840
patch=1,EE,001b69b0,word,461e0000
patch=1,EE,001b69b4,word,461df782
patch=1,EE,001b69b8,word,461dffc2
patch=1,EE,001b69bc,word,e60000c4
patch=1,EE,001b69c0,word,0c06fb52
patch=1,EE,001b69c4,word,e60100b0