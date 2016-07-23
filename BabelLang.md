babel keyboard language family.

# Goal:
Short: A better replacement for on screen keyboards. 
Long:Ability to enter text on a device without a keyboard with precision and speed like that of a keyboard. 
Must enable high precision, high speed, hands free text entry. 
It could also be used as a reader program for the blind, a listener program for the deaf, or a speaker program for the dumb.
It could be used in surgery rooms, factories, and restaurant kitchens to allow sanitary/clean use of computers without having to sanitize or clean the computer or the user.
 
# Approach:
Create a family of constructed spoken languages with a unique single syllable word for each unique key on the keyboard. 
Each syllable starts with a consonant sound and ends with a vowel sound. This eliminates the need for pauses to delineate syllables and allows syllables to be unambiguously identified even if they start to run together.
For each language in the family create speech to text input software.
For each language in the family create text to speech software. This should greatly mitigate the difficulty of learning a language by allowing users to type and hear the syllables of the keys they are typing.
There may be use for a language that allows users with any native language to select a language/keyboard layout. Applications using this language may have to prompt the users in their native language similarly to the 'Decir Español para seleccionar español' that we currently have on various voice menu systems. What languages it prompts for will probably vary with installation or region. 
Each language should have a command that exits and returns microphone control to whatever scope invoked it. For example, if you used the language selection language to select babel qwerty en-us English and speak the exit command babel qwerty will exit and return control to the language selection language. If babel qwerty was activated by a command by the mouse and you speak the exit command the speech recognition engine will exit and the microphone will be turned off. If the language selection language was activated by the platforms native voice command software and the exit command is given babel qwerty will exit and the native voice command software will be turned back on.

# Languages
The description of babel qwerty en-us English is here (link) with a table of key/sound mappings here (link).
Other languages would be devised for other keyboard layouts, character sets, and phoneme sets. (Users that speak different languages will find some syllables easier to pronounce and differentiate. A language for Japanese speaking users would probably use only one of the phonemes corresponding to L or R in English. A language for Chinese speaking users may use tonal components. A language for German speaking users may use more guttural phonemes. A language for French speaking users may use nasality to distinguish between syllables.) 
Thoughts with regards to mouse or touchscreen device replacement are here (link).
A language of easy/fast gestures could be devised for signing people. This may be pointless compared to just using a keyboard or the existing signs for letters. 

# Implementation steps
## First Implementation
Voice to text application for Windows desktop. 
Goal: Test it out in an easy development setting. Look for low hanging fruit in the ergonomics
Make use of the Windows Speech Recognition and Windows Speech Synthesis apis for the heavy lifting. 
Perhaps use VoCoLa (vocola.net) as a starting codebase.
### Key Speaker mode. 
For each key pressed says the corresponding syllable.
### Text Speaker mode.
Provides a text box. When a button is clicked says the syllables for the text in the box.
suggested forms of first implementation.
### Keyboard Listener mode.
Listens on the microphone and outputs keyboard actions via a virtual keyboard.
## First Application
Android keyboard extension or keyboard replacement. 
Have a open source version available for free, but put a $1-5 version in the app stores. 
Make use of open source speech recognition and speech synthesis libraries for the heavy lifting. Can't use GPL or LGPL software to distribute in the app stores. Don't want to make use of Google apis so that we can work offline and cross platform. 
Later make a version for IOS and others.
### Normal Onscreen Keyboard
Try to use Googles keyboard or the Android keyboard for this.
### Read To Me button/command
Speaks the syllables of the currently selected text or the text in the currently selected text box. 
### Listen To Me button
Listens on the microphone and outputs keyboard actions.

# Detailed goals:
Ideally this could be used hands free and eyes free for general purpose text entry by normal users with a minimum of previous learning on any device, with or without an internet connection. 
    - You should be able to use this without an internet connection.
    - You should be able to use this with any application that uses a keyboard.
    - You should be able to use this on weak devices.
    - You should be able to input text quickly.
    - You should be able to input any type of text, not just what the software creator expects.
    - You should be able to input text accurately. You should not have to check what the device heard. You should be able to easily predict what it will do from any given input.
    - You should be able to get fast feedback on what the device heard without having to look at it.
    - You should not have to touch the device.
    - You should not have to learn a large vocabulary to use this.
    - The vocabulary should be structured to make it easy to learn. 
    - The vocabulary should be easy to pronounce.
    - The software should help with learning the vocabulary, via speech synthesis or other features.

# Inspiration/Previous Works and their disadvantages: 
    - http://www.benkuhn.net/autocomplete
    Particularly this section:
    > First of all, the keyboard is a much higher-bandwidth interface than pointing and clicking. If you type 60 words per minute you’re typing 4 characters per second, for 24 bits of entropy per second. To match that precision with a mouse would require you to be able to identify and navigate to any thumbnail-sized part of your screen in a third of a second.6 Maybe barely doable if you’re very dextrous.
    - https://en.wikipedia.org/wiki/Solmization
        - via https://en.wikipedia.org/wiki/Solf%C3%A8ge
            - via https://en.wikipedia.org/wiki/Do-Re-Mi
    - NATO phonetic alphabet. 
        - It doesn't hit enough of the keyboard and has two syllable words for all the supported characters.
    - The English spoken alphabet. 
        - Many words sound like each other, espcially over a noisy connection (such as a phone microphone in a bar or over a phone line). (D vs E vs T, S vs F)
        - Slow. Need to say things like 'Capitol', 'semicolon', 'right parenthesis'
        - No distinguising between input and meta intput. Is the phrase 'capital x' telling the listener to enter 'X' or 'capital x'?
    -VoCoLa vocola.net
        - Specialized.
        - User has to know a lot of words to use well.
        - Results of imput are not easily predictable.
        - Doesn't work with all software.
        - Not cross platform.
    - IPA https://en.wikipedia.org/wiki/International_Phonetic_Alphabet
    - https://www.youtube.com/watch?v=8SkdfdXWYaI
        - Very impressive. Very cool. 
        - Even more specialized than VoCoLa.
        - User has to know a lot more words to use well.
        - Words do not map intuitively to actions. 
        - Still not perfectly reliable. 
        - Not cross platform.
    - Modern speech recognition on natural languages
        - Rely on machine learning.
        - Rely on massive data collection.
        - Rely on complex expensive lingual models. If your desired results are off model it won't work very well for you.
        - Generally don't work offline or on weak devices.
        - Rely on context and models to distinguish between input text and imput commands (Similar to the English spoken alphabet.)
        - Feedback to the user on what they have input is generally clumsy.
    - Phone voice menus. 
        - Only works well on a very small set of expected input. 
    - ROILA http://roila.org/
    - An old Sci-Fi story about computers with an audible machine to machine interface. After being exposed to the noises of the computers talking to each other for a while the people began to make music similar to the interface noises and later to understand and 'speak' the interface noises. 
    - A story from http://www.catb.org/jargon/html/ about a time when computers were big/loud enough that engineers could diagnose problems just by listening to the noises they would make. 
    - Stories in other disciplines where users could work blind just by listening. 
        - https://www.reddit.com/r/talesfromtechsupport/comments/38wpkt/i_can_do_it_blindfoldedgo_on_then)
    - Azimovs ideas about a C-Fe culture.
    
# Name
Named babel due to the initial experiments with speaking it sounding like babbling. 
A second explanation is that it is a tribute to the myth/legend/bible story of the tower of babel.



<br/><p align="center"><a rel="license" href="http://creativecommons.org/licenses/by/4.0/">
<img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" />
</a></p>
