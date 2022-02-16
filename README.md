# MensaUnisa/Cacher
A simple set of scripts that converts University of Salerno's refectory's online
published menus to an sqlite database and also downloads them in three different 
formats.

Run by running `./execute` inside the directory, or put it in your crontab to 
always keep it updated.

All data downloaded is, obviously, in Italian.

## Non-POSIX Dependencies
The software should run with both GNU and BusyBox coreutils, plus
the following:
* `curl`
* `perl`
* poppler-utils (specifically, `pdftotext`, and `pdftoppm` if running with
the `CONVERT_PNG` option)
* Liberation fonts or Arial and Times New Roman fonts (if running with 
`CONVERT_PNG` option)

## Variables
`STARTING_ID`: Files for the UniSa's refectory are stored on the ADiSURC server 
with an unique numerical ID. The software scrapes all of them starting from either
the latest ID processed or `$STARTING_ID` if it's the first time it's run. By default
this is 3335, but it'll get probably updated when file number 3335 becomes too old.

`DATADIR`: The directory where you're storing the csv database and the `files` subdirectory 
containing the various files downloaded and converted. Default value is `$HOME/adisurc`.

`CONVERT_PNG`: Enables PNG conversion if set to any value. It is toggleable because it requires
the installation of extra fonts and may consume unneeded space.

## Storage Formats
The menus are stored in the following formats (`id` is used here in place of the file's
ID on the ADiSURC website):

### Raw PDF (`id`.pdf)
This is the PDF as it's stored on the ADiSuRC's servers. Currently it means it gets 
converted from a (probably) manually-generated Word document, but this might change
in the future. We'll take care of that when we get there.

### PNG (`id`.png)
This is the PDF file converted to PNG through `pdftoppm`.

### Text (`id`.txt)
This is the PDF file converted to a text file in the following format:
```
First servings
---
Second servings
---
Third servings ("Contorno"s)
---
Contents of the Take-Away Basket
```
Every one of these servings is a list of possible meals for that serving, separated by a newline
(`\n`).

### CSV
The data is also stored in two semicolon-separated CSV tables, stored in `$DATADIR`.

#### `menus.csv`
This table contains data about the individual menus.

Each row has the following three columns, in order:
* `id` is the numeric `id` of the file that corresponds to the menu on the ADiSURC servers. This is supposed 
to be unique;
* `date` is the date corresponding to the menu, in YYYYMMDD format;
* `meal` is the specific meal represented. It can be `0` for lunch, and `1` for dinner

Note that there might be multiple menus for the same day (the date/meal combo is NOT guaranteed 
to be unique). This is because the ADiSURC uploads corrections and changes to the menus as 
entirely new files. As such, it's suggested that if you need a specific day's menu you fetch the
one with the highest id for the specified date.

#### `dishes.csv`
This table contains data about the individual dishes server.

Each row has the following three columns, in order:
* `menu` is the numeric `id` of the menu this dish is part of;
* `serving` is the specific serving represented. `0-2` for First, Second and Third serving, `3` for the
contents of the takeaway basket;
* `contents` is the actual dish as described on the menu

## Dockerfile
You may run this through Docker by mounting a volume to `/adisurc` (or passing your own `DATADIR` 
environment variable and mounting the volume there). The Dockerfile is extremely simple and based
on the `alpine` image. It contains the `ttf-liberation` package and as such supports the 
`CONVERT_PNG` operation.

## License Compliance
This is the official stance of the ADiSURC, in Italian, on using their website content:

> I file presenti in questo sito per lo scaricamento (download) quali ad esempio la modulistica sono
liberamente e gratuitamente disponibili. La riproduzione o l'impiego di informazioni testuali e
multimediali (suoni, immagini, software ecc.) sono consentiti con indicazione della fonte e, qualora
sia richiesta un'autorizzazione preliminare, questa indicherà esplicitamente ogni eventuale 
restrizione.

As such, if you use any file generated by this application, it is suggested that you specify the
source as "Azienda per il Diritto allo Studio Universitario della Regione Campania via adisurcampania.it".
