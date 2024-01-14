# UofTCTF 2024 

I'll break down the steps to solve **Secret Message 2 - Forensics**  challenge.


# Challenge Description

The super secret organization changed their flag again. Can you work your magic again?

	> Hint: The flag characters contain abcdefghijklmnopqrstuvwxyz_

## Solution

When we look at the given `png` file, we notice the redacted, pixelated text.
<img src="https://github.com/HashemSalhi/CTF-Writeups/blob/main/UofTCTF%202024/Forensics/Secret%20Message%202/Screenshots/Screenshot1.png">

There is a tool called **[Unredacter](https://github.com/bishopfox/unredacter)** that can uncover redacted, pixelated text to reveal the original text.
This blog provides extensive information about the tool https://bishopfox.com/blog/unredacter-tool-never-pixelation.

I'll follow the the tool manual on how to use it.

### Cropping the image
I'll be using GIMP as recommended in the tool repo, we need to crop the image down to just the pixelated area.
Notice that each pixelated block size is `8x8 px` .
<img src="https://github.com/HashemSalhi/CTF-Writeups/blob/main/UofTCTF%202024/Forensics/Secret%20Message%202/Screenshots/Screenshot2.png">
### Editing the tool configurations
Lets take a look on the character set the tool uses, which in the `preload.ts` file,
It's `abcdefghijklmnopqrstuvwxyz ` 
<img src="https://github.com/HashemSalhi/CTF-Writeups/blob/main/UofTCTF%202024/Forensics/Secret%20Message%202/Screenshots/Screenshot3.png">
Going back to the challenge description the hint states that there is an underscore in the flag characters so let's add it to `guessable_characters` to look like this :

<img src="https://github.com/HashemSalhi/CTF-Writeups/blob/main/UofTCTF%202024/Forensics/Secret%20Message%202/Screenshots/Screenshot4.png">
### Running the tool
We are now ready to start the tool. Run `npm start` and the tool gui will pop up.
<img src="https://github.com/HashemSalhi/CTF-Writeups/blob/main/UofTCTF%202024/Forensics/Secret%20Message%202/Screenshots/Screenshot5.png">
Hit the **Click to start** button.
The tool is stuck on the first character :(
<img src="https://github.com/HashemSalhi/CTF-Writeups/blob/main/UofTCTF%202024/Forensics/Secret%20Message%202/Screenshots/Screenshot6.png">
### Troubleshooting
I spent some time on this, the problem is with our cropped image, I noticed a small difference in colors using a color selector in the block before and upward of the pixelated area, the whole page white color is `#ffffff`:
<img src="https://github.com/HashemSalhi/CTF-Writeups/blob/main/UofTCTF%202024/Forensics/Secret%20Message%202/Screenshots/Screenshot7.png">
 But the block before and upward of the pixelated area contains some other white shades like `#fefefe` and `#fdfdfd` , that means that it's a part of the pixelated area
<img src="https://github.com/HashemSalhi/CTF-Writeups/blob/main/UofTCTF%202024/Forensics/Secret%20Message%202/Screenshots/Screenshot8.png">
So I cropped the image again with one block before and upward of the previous pixelated area, so it looks like this:
<img src="https://github.com/HashemSalhi/CTF-Writeups/blob/main/UofTCTF%202024/Forensics/Secret%20Message%202/Screenshots/Screenshot9.png">
### Getting the Flag
Now lets run the tool again:
<img src="https://github.com/HashemSalhi/CTF-Writeups/blob/main/UofTCTF%202024/Forensics/Secret%20Message%202/Screenshots/Screenshot10.png">
**uoftctf{pokemon_catching_ezz}**
