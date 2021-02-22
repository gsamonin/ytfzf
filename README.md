# ytfzf

A posix script that helps you find Youtube videos (with out API) and opens/downloads using mpv/youtube-dl.
* History support
* Download support
* Format selection (and default selection)

Initially this used to be a single line script. But for portability and extensibility I am breaking my vow. If you still are here for the memes then use the line below. 

	#!/bin/sh
	[ -z "$*" ] || curl "https://www.youtube.com/results" -s -G --data-urlencode "search_query=$*" |  pup 'script' | grep  "^ *var ytInitialData" | sed 's/^[^=]*=//g;s/;$//' | jq '..|.videoRenderer?' | sed '/^null$/d' | jq '.title.runs[0].text,.longBylineText.runs[0].text,.shortViewCountText.simpleText,.lengthText.simpleText,.publishedTimeText.simpleText,.videoId'| sed 's/^"//;s/"$//;s/\\"//g' | sed -E -n "s/(.{60}).*/\1/;N;s/\n(.{30}).*/\n\1/;N;N;N;N;s/\n/\t|/g;p" | column -t  -s "$(printf "\t")" | fzf --delimiter='\|' --nth=1,2  | sed -E 's_.*\|([^|]*)$_https://www.youtube.com/watch?v=\1_' | xargs -r -I'{}' mpv {}


# Recent Updates

* Defaults can be now set through environment variables instead of changing the script.

* Use external menu (dmenu/rofi) instead of fzf using option -D  (delete history has been shifted to -x), the menu command has to be specified as an environmental variable. 

By default the external menu is `dmenu -i -l 30` . If you want any other menu like rofi then export this variable in your ( ~/.bashrc , ~/.zshrc)

```
export YTFZF_EXTMENU=' rofi -dmenu -fuzzy -width 1500'
```

> You can read more about this below. Dmenu may be slow when dealing with non English characters and symbols. 

* Option to loop the video menu, the menu will be shown again as the video ends/closes.

# Usage

![Gif](ytfzf.gif)

```
Usage: ytfzf <search query>
     -h                    Show this help text
     -D  <search query>    Use dmenu instad of fzf
                           warning: doesn't work as smooth as fzf
     -H                    Choose from history
     -x                    Delete history
     -m  <search query>    Audio only (for listening to music)
     -d  <search query>    Download to current directory
     -f  <search query>    Show available formats before proceeding
     -a  <search query>    Auto play the first result, no fzf
     -l  <search query>    loop: prompt again after video ends
 ```


* Videoes can be selected using fzf (default) or dmenu.

## Examples
> Watch video

	ytfzf <query>
	
* You can use multiple options together, here are some examples

> Steam audio (music), and prompt as the music finishes

	ytfzf -ml <query>


> Download a video from your history

	ytfzf -dH

> Open using dmenu in a certain format

	ytfzf -fD  <query>

If you started watching a video and you wish to change format then 
first hit Q to save position and quit mpv, then choose your format using

	ytfzf -faH

## Useful mpv keybindings to keep in mind
* Use `J` for subtitles (also works with audio, if the music video has subtitles)
* Use `L` for single-loop

# Dependencies
* mpv
* [youtube-dl](https://github.com/ytdl-org/youtube-dl)
* [fzf](https://github.com/junegunn/fzf) - for menu
* [jq](https://github.com/stedolan/jq) - to parse json

### Arch based

	sudo pacman -S jq mpv youtube-dl fzf 

### Debian based

	sudo apt install jq mpv youtube-dl fzf 

> Note youtube-dl is usually outdated in debian repos, I suggest getting it from 

* [youtube-dl github](https://github.com/ytdl-org/youtube-dl)

# Installation

	git clone https://github.com/pystardust/ytfzf
	cd ytfzf
	chmod +x ytfzf

Copy it to your path
	
	sudo cp ytfzf /usr/local/bin/

Arch users can install ytfzf from the [AUR](https://aur.archlinux.org/packages/ytfzf-git/)
	
	yay -S ytfzf-git
        

# Defaults

Defaults can be set by environmental variables.

> Environment variables can be set in many ways. You can add them to you ~/.bashrc 

## Format

If you prefer to watch Youtube videos in certain option with out the prompting every single time.

```
export YTFZF_PREF="22"                   
```

> 22: 720p,
> 18: 360p

If the preferred format is not available then, it will go back to auto selection.



## History

On by default. If you don't want history.

```
export YTFZF_HIST=0
```
> 0: history off, 1: history on

* This history will be stored in the cache file as `ytfzf_hst`

You can modify the file location by changing the cache directory

```
export YTFZF_CACHE=~/.cache/ytfzf
```


## Loop menu prompt

Off by default.  Can be turned on using option -l. Or setting

```
export YTFZF_LOOP=1
```

This would return you to the video selection prompt after the video is exited/ends.

> To quit the script you can press `ESC` or `^C` in the video selection prompt.

## External menu command (DMENU / ROFI)

To use an external menu you will need to pass in the `-D` option
```
ytfzf -D <query>
```

By default the external menu is set to dmenu `dmenu -i -l 30`. You can modify to rofi this by

```
export YTFZF_EXTMENU=' rofi -dmenu -fuzzy -width 1500'
```

You also may need to modify the width of the output that is being piped into external menu

```
export YTFZF_EXTMENU_LEN=180
```

> Depending on you screen resolution and font size this may need to be modified

> WARNING : dmenu doesn't behave well with some fonts. Expect it to be slow when you have non-English character and symbols.



## Currently Playing

On by default. Stores the details of the currently playing track. Empty when nothing is playing. This could be used in status bar modules.


To disable it
```
export YTFZF_CUR=0
```

It will be stored in the ytfzf cache directory as `ytfzf_cur`


# Todo

* Playlists
* Icons
