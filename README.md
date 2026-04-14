# Project Hail Mary - Eridanese Translator

A real-time audio translator based on the alien communication system in Andy Weir's _Project Hail Mary_. In the book, the alien Rocky communicates using musical tones that correspond to words. I realized it'd be a pretty simple software to build so I figured might aswell! Only thing it can't do is chords, since that gets quite a bit more advanced. If you want more of an explanation you can checkout the reel I made on it [here!](https://www.instagram.com/reel/DWwzvCPhQCu/?utm_source=ig_web_copy_link&igsh=MzRlODBiNWFlZA==)

## How It Works

1. Audio is captured from an input device in real time (MAKE SURE TO SELECT THE RIGHT `INPUT_DEVICE_INDEX`)
2. A Fast Fourier Transform (FFT) is performed on each audio chunk to find the dominant frequency
3. That frequency is looked up in `vocab.csv` to find the corresponding word
4. Words are collected into a sentence until a pause is detected
5. Claude corrects the broken sentence into "proper" grammar
6. ElevenLabs speaks the result aloud

> NOTE: The fundemental frequency detection I wrote is quite primitive. As you can see, it simply finds the frequency with the highest amplitude and assumes that that's the fundemental frequency. In a lot of cases, this works, but there are many instruments/sound sources where it's not that simple. Because of that, the software works best with simple sound sources such as a sin wav osc or a piano.

## Requirements

- Python 3.x
- A virtual environment (recommended)

Install dependencies:

```
pip install -r requirements.txt
```

## API Keys

This project uses the Claude API (Anthropic) and ElevenLabs. Obviously the Elevenlabs key is needed if you want the real text-to-speech, but I put an optional flag you can add for Claude if you don't care to use it, since it's not really that needed. (You can also turn off elevenlabs and just have it displayed in your terminal!)

Add them as environment variables so you do not have to hardcode them in the script. Add these lines to your `~/.zshrc` (Mac) or `~/.bashrc` (Linux):

```
export ANTHROPIC_API_KEY="your-anthropic-key-here"
export ELEVENLABS_API_KEY="your-elevenlabs-key-here"
```

Then reload your shell:

```
source ~/.zshrc
```

The script will pick up both keys automatically.

You can get an Anthropic API key at console.anthropic.com and an ElevenLabs key at elevenlabs.io.

## Configuration

At the top of `EridianeseTranslator.py` there are several constants you may need to adjust:

- `INPUT_DEVICE_INDEX` - the index of your audio input device. Run the commented-out device listing code in the script to find the right index for your setup
- `AMPLITUDE_THRESHOLD` - how loud a note needs to be before it is detected
- `CHANGING_THRESHOLD` - how many Hz a frequency needs to change before a new note is registered
- `READING_CLEARANCE` - how many +- Hz of tolerance is allowed when matching a frequency to vocab.csv
- `PHRASE_TIME_LIMIT_SECONDS` - how long to wait after the last note before the sentence is sent to Claude
- `ELEVENLABS_VOICEID` - the ID of the ElevenLabs voice to use

## Running the Script

```
python3 EridianeseTranslator.py
```

### Arguments

`--no-ttp` - Disables Claude and ElevenLabs entirely. The detected words will just be printed to the terminal as a joined sentence. Useful for testing frequency detection without making any API calls.

```
python3 EridianeseTranslator.py --no-ttp
```

`--no-claude` - Skips the Claude grammar correction step and feeds the raw detected words directly to ElevenLabs. This will make 'Rocky' sound more like he did in the film! I really just added Claude because I figured in a real world situation, you would want something like that to help you out with grammar.

```
python3 EridianeseTranslator.py --no-claude
```

`--hide-spectrum` - Hides the spectrum graph.

```
python3 EridianeseTranslator.py --hide-spectrum
```

`--hide-freq` - Hides the detected frequencies from being printed in the terminal.

```
python3 EridianeseTranslator.py --hide-freq
```

All arguments can be combined:

```
python3 EridianeseTranslator.py --no-claude --no-ttp --hide-spectrum --hide-freq
```

## vocab.csv

This file maps frequencies (in Hz) to words. Each row has three columns: frequency, word, and note name. Note name is simply for helping you remember what the frequencies are, the program totally ignores it.

Example:

```
392, Hi, G4
440, friend, A4
```

You can expand or modify this file to build out your own vocabulary. The `vocab.csv` file included in this repo is a larger reference mapping of 100+ common words to standard western 12-tone frequencies, laid out as an example of what a full vocabulary could look like. Keep in mind, you'll have to adjust the `CHANGING_THRESHOLD` & `READING_CLEARANCE` constants if you're using low frequencies, or else the changes won't be registered.

> I also included most of the csv files I used in my reel so you can view those as an example of mapping a specific phrase/sentence.

Use the displayed frequencies while playing notes to find the exact frequencies your instrument or setup produces, then add them to `vocab.csv`.

## Stopping the Script

Press `Ctrl+C`. The audio stream will be cleanly closed automatically.

> Fun fact when writing this I forget to have it close the streams when I hit Ctrl C so I ended up having the audio streams take up 80gb of memory and slowly crash my Mac. Won't happen anymore though!
