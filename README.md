# JP-study

- [Anki Mining Template](#anki-mining-template)
- [Yomichan settings](#yomichan-settings)
    - [Fields](#fields)
    - [Custom Card Templates (Handlebars)](#custom-card-templates-handlebars)
    - [Popup Appearance custom CSS](#popup-appearance-custom-css)
- [Other Templates](#other-templates)
    - [Kanken Deck](#kanken-deck-link)

## Anki Mining Template

[Link](https://drive.google.com/drive/u/2/folders/1et5YNL6pGRwELfWSTRpIs-pwFVX74Nbq)

<p align="center">
    <kbd><img src="images/vocab_back.png" width="70%" /></kbd>
    <kbd><img src="images/sentence_back.png" width="70%" /></kbd>
    <kbd><img src="images/mining_template.gif" width="70%" /></kbd>
</p>


## Yomichan settings

### Fields
| Field | Value |
|----------|----------|
| Expression | `{expression}` |
| Expression (furigana) | `{furigana-plain}` | 
| Expression (reading) | `{reading}` |
| Expression (audio) | `{audio}` | 
| MainDefinition | `{main-def}`|
| Sentence | `{cloze-prefix}<b>{cloze-body}</b>{cloze-suffix}` | 
| Sentence (furigana) |  |
| Sentence (audio) |  | 
| FullDefinition | `{glossary}` |
| Image |  | 
| Translation |  |
| PitchPosition | `{pitch-accent-positions}` |
| Hint |  |
| Frequency | `{frequencies}` | 
| FreqSort | `{freq}` |
| MiscInfo | `{document-title}` | 
| ExtraField | `{jlpt} {ln}` |
| *IsSentenceCard | `{grammar-pt}` | 

Notes:
- When **\*IsSentenceCard** isn't empty, card is turned into a sentence card. When empty, it's a vocab card.
- **PitchPosition** takes `{pitch-accent-positions}`. `{pitch-accents}` and `{pitch-accent-graphs}` do not work.
- `{main-def}`, `{freq}`, `{jlpt}`, `{ln}`, and `{grammar-pt}` are custom templates/handlebars
- **Hint** is for a hint on the front of the card.
- **Sentence (furigana)** is for furigana generated from [AJT Furigana](https://ankiweb.net/shared/info/1344485230) (Anki addon)
- **Sentence (audio)**, **Image**, **Translation** are for data from [Mpvacious](https://github.com/Ajatt-Tools/mpvacious). **Sentence**, **MiscInfo** are overwritten by data from Mpvacious as well.
- **ExtraField** is for space-separated tags to be generated through [Field to Tag](https://ankiweb.net/shared/info/1600845494) (Anki addon)

### Custom Card Templates (Handlebars)

- `{freq}` - frequency value for sorting new cards. [Link](https://github.com/MarvNC/JP-Resources#sorting-mined-anki-cards-by-frequency).

```handlebars
{{#*inline "freq"}}
    {{~! Frequency sort handlebars: v23.02.05.1 ~}}
    {{~! The latest version can be found at https://github.com/MarvNC/JP-Resources ~}}
    {{~#scope~}}
        {{~! Options ~}}
        {{~#set "opt-ignored-freq-dict-regex"~}} ^(JLPT_Level)$ {{~/set~}}
        {{~#set "opt-keep-freqs-past-first-regex"~}} ^()$ {{~/set~}}
        {{~set "opt-no-freq-default-value" 0 ~}}
        {{~set "opt-freq-sorting-method" "harmonic" ~}} {{~! "min", "first", "avg", "harmonic" ~}}
        {{~! End of options ~}}

        {{~! Do not change the code below unless you know what you are doing. ~}}
        {{~set "result-freq" -1 ~}} {{~! -1 is chosen because no frequency dictionaries should have an entry as -1 ~}}
        {{~set "prev-freq-dict" "" ~}}
        {{~set "t" 1 ~}}

        {{~#each definition.frequencies~}}

            {{~! rx-match-ignored-freq is not empty if ignored <=> rx-match-ignored-freq is empty if not ignored ~}}
            {{~#set "rx-match-ignored-freq" ~}}
                {{~#regexMatch (get "opt-ignored-freq-dict-regex") "gu"~}}{{this.dictionary}}{{~/regexMatch~}}
            {{/set~}}
            {{~#if (op "===" (get "rx-match-ignored-freq") "") ~}}

                {{~!
                    only uses the 1st frequency of any dictionary.
                    For example, if JPDB lists 440 and 26189㋕, only the first 440 will be used.
                ~}}
                {{~set "read-freq" false ~}}
                {{~#if (op "!==" (get "prev-freq-dict") this.dictionary ) ~}}
                    {{~set "read-freq" true ~}}
                    {{~set "prev-freq-dict" this.dictionary ~}}
                {{/if~}}

                {{~#if (op "!" (get "read-freq") ) ~}}
                    {{~#set "rx-match-keep-freqs" ~}}
                        {{~#regexMatch (get "opt-keep-freqs-past-first-regex") "gu"~}}{{this.dictionary}}{{~/regexMatch~}}
                    {{/set~}}

                    {{~! rx-match-keep-freqs is not empty if keep freqs ~}}
                    {{~#if (op "!==" (get "rx-match-keep-freqs") "") ~}}
                        {{~set "read-freq" true ~}}
                    {{/if~}}
                {{/if~}}

                {{~#if (get "read-freq") ~}}
                    {{~set "f" (op "+" (regexMatch "\d+" "" this.frequency)) ~}}

                    {{~#if (op "===" (get "opt-freq-sorting-method") "min") ~}}
                        {{~#if
                            (op "||"
                                (op "===" (get "result-freq") -1)
                                (op ">" (get "result-freq") (get "f"))
                            )
                        ~}}
                            {{~set "result-freq" (op "+" (get "f")) ~}}
                        {{~/if~}}

                    {{~else if (op "===" (get "opt-freq-sorting-method") "first") ~}}
                        {{~#if (op "===" (get "result-freq") -1) ~}}
                            {{~set "result-freq" (get "f") ~}}
                        {{~/if~}}

                    {{~else if (op "===" (get "opt-freq-sorting-method") "avg") ~}}

                        {{~#if (op "===" (get "result-freq") -1) ~}}
                            {{~set "result-freq" (get "f") ~}}
                        {{~else~}}
                            {{~!
                                iterative mean formula (to prevent floating point overflow):
                                    $S_{(t+1)} = S_t + \frac{1}{t+1} (x - S_t)$
                                - example java implementation: https://stackoverflow.com/a/1934266
                                - proof: https://www.heikohoffmann.de/htmlthesis/node134.html
                            ~}}
                            {{~set "result-freq"
                                (op "+"
                                    (get "result-freq")
                                    (op "/"
                                        (op "-"
                                            (get "f")
                                            (get "result-freq")
                                        )
                                        (get "t")
                                    )
                                )
                            }}
                        {{~/if~}}
                        {{~set "t" (op "+" (get "t") 1) ~}}

                    {{~else if (op "===" (get "opt-freq-sorting-method") "harmonic") ~}}
                        {{~#if (op ">" (get "f") 0) ~}} {{~! ensures only positive numbers are used ~}}
                            {{~#if (op "===" (get "result-freq") -1) ~}}
                                {{~set "result-freq" (op "/" 1 (get "f")) ~}}
                            {{~else ~}}
                                {{~set "result-freq"
                                    (op "+"
                                        (get "result-freq")
                                        (op "/" 1 (get "f"))
                                    )
                                }}
                                {{~set "t" (op "+" (get "t") 1) ~}}
                            {{~/if~}}
                        {{~/if~}}

                    {{~else if (op "===" (get "opt-freq-sorting-method") "debug") ~}}

                        {{ this.dictionary }}: {{ this.frequency }} -> {{ get "f" }} <br>

                    {{~else~}}
                        (INVALID opt-freq-sorting-method value)
                    {{~/if~}}

                {{~/if~}}

            {{~/if~}}

        {{~/each~}}

        {{~! (x) >> 0 apparently floors x: https://stackoverflow.com/a/4228528 ~}}
        {{~#if (op "===" (get "result-freq") -1) ~}}
            {{~set "result-freq" (get "opt-no-freq-default-value") ~}}
        {{~ else if (op "===" (get "opt-freq-sorting-method") "avg") ~}}
            {{~set "result-freq"
                (op ">>" (get "result-freq") 0 )
            ~}}
        {{~ else if (op "===" (get "opt-freq-sorting-method") "harmonic") ~}}
            {{~set "result-freq"
                (op ">>"
                    (op "*"
                        (op "/" 1 (get "result-freq"))
                        (get "t")
                    )
                    0
                )
            ~}}
        {{~/if~}}

        {{~get "result-freq"~}}
    {{~/scope~}}
{{/inline}}
```

- `{main-def}` - for the **MainDefinition** field. If text is selected, return `{selection-text}` otherwise return `{jmdict-def}` (English/JMDict definition).
```handlebars
{{~#*inline "jmdict-def"~}}
    <div style="text-align: left;">
    {{~#scope~}}
        {{~#if (op "===" definition.type "term")~}}
            {{~#if (op "===" this.dictionary "JMdict (English)")~}}
                {{~> glossary-single definition brief=brief noDictionaryTag=noDictionaryTag ~}}
            {{/if}}
        {{~else if (op "||" (op "===" definition.type "termGrouped") (op "===" definition.type "termMerged"))~}}
            {{~#set "jmdict-length" 0}}{{/set~}}
            {{~#each definition.definitions~}}
                {{~#if (op "===" this.dictionary "JMdict (English)")~}}
                    {{~#set "jmdict-length" (op "+" (get "jmdict-length") 1) ~}}{{~/set~}}
                {{~/if~}}
            {{~/each~}}

            {{~#if (op ">" (get "jmdict-length") 1)~}}
                <ol>{{~#each definition.definitions~}}{{~#if (op "===" this.dictionary "JMdict (English)")~}}<li>{{~> glossary-single . brief=../brief noDictionaryTag=../noDictionaryTag ~}}</li>{{/if}}{{~/each~}}</ol>
            {{~else~}}
                {{~#each definition.definitions~}}{{~#if (op "===" this.dictionary "JMdict (English)")~}}{{~> glossary-single . brief=../brief noDictionaryTag=../noDictionaryTag ~}}{{/if}}{{~/each~}}
            {{~/if~}}
        {{~/if~}}
    {{~/scope~}}
    </div>
{{~/inline~}}

{{~#*inline "main-def"~}}
    {{~#set "selected"}}{{~> selection-text}}{{/set~}}
    {{~#set "pattern1"~}}
        [
        {{~#regexReplace "(<span class=\"term\">)|(</span>)|(<ruby>)|(</ruby>)|(<rt>)|(</rt>)|[\[\]]" "" "g"~}}
        {{~> cloze-body}}
        {{~/regexReplace~}}
        ]
    {{~/set~}}
    {{~#set "pattern2"~}}
        [
        {{~#regexReplace "(<span class=\"term\">)|(</span>)|(<ruby>)|(</ruby>)|(<rt>)|(</rt>)|[\[\]]" "" "g"~}}
        {{~#getMedia "textFurigana" definition.cloze.body escape=false}}{{~/getMedia~}}
        {{~/regexReplace~}}
        ]
    {{~/set~}}
    {{~#set "diff1"}}{{~#regexReplace (get "pattern1") ""~}}{{~#get "selected"}}{{/get~}}{{~/regexReplace~}}{{/set~}}
    {{~#set "diff2"}}{{~#regexReplace (get "pattern2") ""~}}{{~#get "selected"}}{{/get~}}{{~/regexReplace~}}{{/set~}}

    {{~#if (op "&&"
                (op ">" (property (get "diff1") "length") (op "-" (property (get "selected") "length") (property (get "diff1") "length")))
                (op ">" (property (get "diff2") "length") (op "-" (property (get "selected") "length") (property (get "diff2") "length")))
            )}}
        {{~> selection-text}}
    {{~else~}}
        {{~> jmdict-def noDictionaryTag=true brief=true ~}}
    {{/if~}}
{{~/inline~}}
```

- `{jlpt}` - space-separated jlpt tags for the [Field to Tag](https://ankiweb.net/shared/info/1600845494) add-on.

```handlebars
{{~#*inline "jlpt"~}}
    {{~#if (op ">" definition.frequencies.length 0)~}}
        {{~#each definition.frequencies~}}
            {{~#if (op "===" this.dictionary "JLPT_Level")~}}
                JLPT_{{frequency~}}
                {{#unless @last}} {{/unless}}
            {{~/if~}}
        {{~/each~}}
    {{~/if~}}
{{/inline}}
```

- `{ln}` - "ラノベ" and the LN title tags for the [Field to Tag](https://ankiweb.net/shared/info/1600845494) add-on when on `reader.ttsu.app`.

```handlebars
{{#*inline "ln"}}
    {{~#set "ttsu" ~}}
        {{~#regexMatch "reader\.ttsu\.app"~}} {{definition.url}} {{~/regexMatch~}}
    {{/set~}}
    {{~#if (op "!==" "" (get "ttsu"))~}}
        ラノベ 
        {{#regexReplace "　" "_" "g"~}}
            {{~#regexReplace "\| ッツ Ebook Reader" ""~}}{{~context.document.title~}}{{/regexReplace~}}
        {{/regexReplace~}}
    {{~/if~}}
{{/inline}}
```

- `{grammar-pt}` - fills out **\*IsSentenceCard** Field when a grammar dictionary has an entry.

```handlebars
{{#*inline "grammar-pt"}}
    {{~#set "is-grammar" false}}{{/set~}}
    {{~#each definition.definitions~}}
        {{~#if (op "||" (op "===" this.dictionary "JLPT文法解説まとめ") (op "===" this.dictionary "日本語文法辞典(全集)"))~}}
            {{~#set "is-grammar" true ~}}{{~/set~}}
        {{~/if~}}
    {{~/each~}}
    {{~#if (op "===" (get "is-grammar") true)~}}x{{/if}}
{{/inline}}
```

### Popup Appearance custom CSS

<img src="images/popup.png" width="60%" />
<kbd><img src="images/search.png" width="100%" /></kbd>

```css
body {
   font-family: IPAexGothic;
}

.source-text {
    font-family: UD Digi Kyokasho N-R;
}

:root[data-theme="dark"] {
    --text-color: #ffffff;
    --background-color: #0d1117;
    --accent-color: #2D4446;
    --accent-color-lighter: #416265;
    --tag-pronunciation-dictionary-background-color: #2d3746;
    --tag-dictionary-background-color: #2F2D46;
    --tag-frequency-background-color: #2D4446;
    --tag-default-background-color: #51647E;
    --tag-name-background-color: #3d4993;
    --tag-expression-background-color: #4857AE;
    --tag-popular-background-color: #232d5a;
    --tag-frequent-background-color: #303e7c;
    --tag-archaism-background-color: #533642;
    --tag-part-of-speech-background-color: #303e7c;
    --input-background-color: #24292f;
    --link-color: #3d4993;
}

/* Fix quotes (https://aquafina-water-bottle.github.io/jp-mining-note/jpresources/#ensuring-properly-quotes-the-text) */
.jp-quote-text {
    text-indent: -1em;
    padding-left: 1em;
}

/* Only show NHK pitch when アクセント辞典 doesn't have data */
:not(ol[data-count='1']) > li.pronunciation-group[data-dictionary='NHK'] {
    display: none;
}
ol.pronunciation-group-list[data-count='2'] {
    list-style: none;
    padding: 0;
}

/* Only show JMDict on hover */
.definition-list li.definition-item[data-dictionary='JMdict (English)'] .gloss-list {
    opacity: 0;
}
.definition-list:has(li.definition-item[data-dictionary='JMdict (English)']:hover) .gloss-list {
    opacity: 1;
}

/* Disable furigana selection */
ruby rt {
    user-select: none;
}
```

## Other Templates

### Kanken Deck ([link](https://ankiweb.net/shared/info/759825185))

<p align="center">
    <kbd><img src="images/kanken_back.png" width="70%" /></kbd>
</p>

- Front

```javascript
<div id="content"> 
    Kanken Level: {{KankenLevel}}
    <div class="sentence_front">
        {{SentenceFront}}
    </div>
    {{#Picture}}
        {{Picture}}
        <br>
    {{/Picture}}	
    {{KankenAudio}}
    <hr>
    <div id="box">
        <div class='vert'></div>
        <div class='hori'></div>
        <div id="diagram" style="opacity: 0;">
            {{Diagram}}
        </div>
    </div>
</div>

<script>
    function makeGrid() {
        const tategaki = true;		/* toggle vertical writing */

        const diagram = document.getElementById('diagram');
        const box = document.getElementById('box');
        let size = 140 * diagram.childElementCount;
        let x = 140;

        if (tategaki) {
            const long = document.getElementsByClassName('vert')[0];
            box.style.height = `${size}px`;
            long.style.height = `${size}px`;
            while (x < size) {
                const short = document.createElement('div');
                short.classList.add('hori');
                short.style.height = `${x}px`;
                box.insertBefore(short,diagram);
                x += 70;
            }
        } else {
            const long = document.getElementsByClassName('hori')[0];
            box.style.width = `${size}px`;
            long.style.width = `${size}px`;
            while (x < size) {
                const short = document.createElement('div');
                short.classList.add('vert');
                short.style.width = `${x}px`;
                box.insertBefore(short,diagram);
                x += 70;
            }
        }
        return;
    }
    makeGrid();
</script>
```

- Back

```javascript
<div id="content"> 
    Kanken Level: {{KankenLevel}}
    <div class="sentence_front">
        {{SentenceBack}}
    </div>
    {{#Picture}}
        {{Picture}}
        <br>
    {{/Picture}}
    {{KankenAudio}}
    <hr>
    <div id="box">
        <div class='vert'></div>
        <div class='hori'></div>
        <div id="diagram">
            {{Diagram}}
        </div>
    </div>
    <div id="extra">
        {{Kana}}【{{Kanji}}】
        <br>
        {{Meaning}}
    </div>
</div>

<script>
    function makeGrid() {
        const tategaki = true;		/* toggle vertical writing */

        const diagram = document.getElementById('diagram');
        const box = document.getElementById('box');
        let size = 140 * diagram.childElementCount;
        let x = 140;

        if (tategaki) {
            const long = document.getElementsByClassName('vert')[0];
            box.style.height = `${size}px`;
            long.style.height = `${size}px`;
            while (x < size) {
                const short = document.createElement('div');
                short.classList.add('hori');
                short.style.height = `${x}px`;
                box.insertBefore(short,diagram);
                x += 70;
            }
        } else {
            const long = document.getElementsByClassName('hori')[0];
            box.style.width = `${size}px`;
            long.style.width = `${size}px`;
            while (x < size) {
                const short = document.createElement('div');
                short.classList.add('vert');
                short.style.width = `${x}px`;
                box.insertBefore(short,diagram);
                x += 70;
            }
        }
        return;
    }
    makeGrid();
</script>
```

- CSS

```css
.card.nightMode {
    --main-bg: #0d1117;
    --sub-bg: #161b22;
    --main-color: #ffffff;
    --sub-color: #8b949e;
    --grey: rgba(128,128,128, 0.1);
    --main-font: "Yu Mincho";
    font-family: var(--main-font);
    background-color: var(--main-bg);
    color: var(--main-color);
    font-size: 20px;
    text-align: center;
}

#qa {
    display: flex;
    align-items: stretch;
    flex-direction: column;
    min-height: calc(100vh - 40px);
}

@font-face {
    font-family: "Yu Mincho";
    src: local("Yu Mincho"), local("游明朝"), url("_yumin.ttf");
}

@font-face {
    font-family: "IPAExGothic";
    src: local("IPAExGothic"), url("_ipaexg.ttf");
}



/* ----- Front elements ----- */


.sentence_front {font-size: 36px;}

/* PC replay button */
.replay-button {margin-top: -5px;}
.replay-button svg {width: 30px; height: auto;}
.replay-button svg path {fill: var(--main-color); transition: .2s;}
.replay-button svg circle {fill: var(--main-bg); display: none;}
.replay-button:hover svg path {fill: var(--sub-color);}

/* Grid */
#box {
    width: 140px;
    height: 140px;
    margin: auto;
    border-style: solid;
    border-color: var(--grey);
    background-color: rgba(255,255,255,0);
}
.vert {
    position: absolute;
    height: 140px;
    width: 70px;
    margin: auto;
    border-style: none;
    border-right-style: dotted;
    border-color: var(--grey);
}
.hori {
    position: absolute;
    height: 70px;
    width: 140px;
    margin: auto;
    border-style: none;
    border-bottom-style: dotted;
    border-color: var(--grey);
}


/* ----- Back elements ----- */


/* Stroke diagram */
#diagram {line-height: 0;}
#diagram > img {
    height: 140px;
    width: 140px;
    position: relative;
    z-index: 100;
}


/* Extra info */
#extra {
    opacity: 0;
}

#extra:hover {
    opacity: 1;
}


/* ---------- Misc ---------- */


/* Remove default margins */
* {
    margin: 0px;
    padding: 0px;
}


/* Images */
img {
    height: 200px;
    width: auto;
    border-radius: 8px;
}


/* Underline CSS */
u {
    text-decoration: none;
    color: #c19fff;
    font-weight: 400;
}
u > ruby rt {
    opacity: 1;
}


/* Line margins */
hr {
    margin-top: 0.5em;
    margin-bottom: 0.5em;
}


```