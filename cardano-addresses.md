# cardano-addresses

The `cardano-addresses` utility is a command line tool for manipulating - guess
what - addresses.

You can find its source code in the following repository: 
https://github.com/input-output-hk/cardano-addresses,
but most users would simply want the compiled binaries from

## Installation

Simply download the latest released file for your operating system for the 
[release page](https://github.com/input-output-hk/cardano-addresses/releases)

Unzip the file and you should get a `cardano-address` binary.

For the sake of simplicity, create a `dev/cardano` folder under your home 
directory, then move the binary into it:

```shell script
mkdir ~/dev
mkdir ~/dev/cardano
mv ~/Downloads/cardano-address ~/dev/cardano
```

Make it executable:
```shell script
cd ~/dev/cardano
chmod +x cardano-address
```  

See if it works
```shell script
./cardano-address --version
```

This should output something like:

```
2.0.0 @ 9a64226fd749513ad0ed2e43e663f7132c92faa7
```

Once you get there, you are ready to move on to learning how to use this tool. 

## The command line interface

IOG did a pretty good job with their command line tools: every command is
organized as "sub-menus" and every time you type in a command you'll get the 
options available from the path you took. 

For example, from `cardano-address` you'll get the top "menu" options:
 
```shell script
./cardano-address

>Available commands:
  version                  Show the software current version and build revision.
  recovery-phrase          About recovery phrases
  key                      About public/private keys
  address                  About addresses
```

Let's try out the `recovery-phrase` option:

```shell script
./cardano-address recovery-phrase

>Available commands:
  generate                 Generate an English recovery phrase
```

So it looks like we only have the `generate` option from there, let's see what
it does:
```shell script
./cardano-address recovery-phrase generate

>return arrest letter ready seat weird news you wave profit ritual grace cream lawsuit lend alone crisp foil joy gas caught vintage fresh modify
```

Amazing, we got a random 24 words seed phrase for a wallet! That was super easy.

### Making it even easier

You can enable bash to auto-complete the available commands for you with this
command:
 
```shell script
source <( ./cardano-address --bash-completion-script `which ./cardano-address`)
```

Now, you can type `./cardano-address` followed by a tab key press to get the 
list of available commands and their arguments:

```shell script
./cardano-address recovery-phrase <TAB> 

> generate  -h        --help    


./cardano-address recovery-phrase generate <TAB> 
./cardano-address recovery-phrase generate -<TAB>

> -h      --help  --size  
```

Hm so it looks like the `generate` option allows for a `--size` parameter, let's
try this out:

```
./cardano-address recovery-phrase generate --size 9

>work swim render cart amazing valve funny tomato drink
```

Et voila, a 9 word long seed phrase. Easy as pie.

So now that you got the hang of it, let's dive deeper into it.

## Generating seed phrases

To generate a seed phrase, simply run 
`./cardano-address recovery-phrase generate --size <WORD COUNT>`

Where the word count is one of 9, 12, 15, 18, 21 or 24 words, eg:

```shell script
./cardano-address recovery-phrase generate --size 12
>zebra dinosaur galaxy valve kangaroo image omit mystery faith crime normal omit

./cardano-address recovery-phrase generate --size 21
>sentence sail donate pioneer theory suspect mammal dove impulse second narrow animal concert bless design flag glow area blame noble forget
```

### Selecting the appropriate word count to restore the seed in your wallet of choice

 * *Shelley* addresses:

   * For daedalus, use `--size 24`.
   
   * For yoroi, use `--size 15`.
 
 * Byron legacy wallet
   
   * For daedalus, use `--size 12`.

   * For yoroi use `--size 15`.

> **NOTE:** There is no support for the legacy Byron paper wallet format, which used 27 words.
 
To make the upcoming instructions easier to follow and reproduce, store the 
seed phrase in a text file. We'll be using `--size 24` from now on.

```
./cardano-address recovery-phrase generate --size 24 > seed.txt
cat seed.txt
>intact engine crumble rotate umbrella flee wire talk creek employ rural state wreck pluck settle later okay foot assault general type mutual excuse valid
```

With the seed phrase *insecurely* stored in a plain text file, we can proceed to the next steps.