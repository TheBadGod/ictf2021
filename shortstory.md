# Short Story

# Task

Do you like my short story? I feel like it's already too long...

farmers quantities beneath that stranger stood looking glasses finished supernatural still in summer indiscriminately befallen tens replied cultivate rear i consider his quadrant white silent undiscriminating scornful tone conveyed which case i happened she screamed partiality follow apprehensiveness captain langsdorff touching that swayed apprehensiveness than moses little apprehensiveness touching a century respectful england burton whereas they entered belong though simultaneousness perhaps circumnavigating blubber hunters should apprehensiveness touching a forehead the keel a peddling from whom they seemed circumnavigating cherish thereby give me chapter hark sideways swung from me that s harpoon untraditionally theory physiognomically regarded among flowers muskiness leading to decipher stubb firmly physiognomically regarded among spouting ubiquitous conclude i know it slipped away without at tranque treacherously drugged conscientious scruples difficulty having irresistibleness wrought ones already we resumed unsurrenderable tricks apprehensiveness forbids due time that parsee physiognomically regarded and somehow before america on captain peleg seeing simultaneousness shadows villainies chapter unconditionally indeed circumnavigating present physiognomically meaning individualizing tidings record nearer indiscriminately befallen tens america conversant knocking going messages water because tweedledum replied unprecedentedly dragged clapping common circumnavigating pequod particularly his perpendicularly all perpendicularly and indomitableness whales understandings persuade uncontaminated

## Solution

After reading the description I thought it had to do with the lengths of the
words, so I put the text in a string and split by spaces, then converted
the length of each word into a new list. I first thought I needed to 
check if a word is a certain length (Because `too long`) and then get a 
binary string (long word vs not long word), but that was not quite correct.

After playing with the lengths for a bit I noticed that all the lengths were
below 17, so I just subtracted one from each length, converted that to hex and
then concatenated all the hex characters to get the flag.

Here a neat oneliner to do that:

`bytes.fromhex("".join([hex(len(i)-1)[2:] for i in open("story.txt").read().strip().split(" ")]))`

And here the flag:

`ictf{A_sh0rt_st0ry_is_4_piece_of_pr0s3_f1ct10n_that_typ1cally_can_b3_read_in_one_sitting_[...]}`
