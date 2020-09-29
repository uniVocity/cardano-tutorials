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

With the command above your seed phrase will be *insecurely* stored in a plain 
text file named `seed.txt`. With this we can proceed to the next steps.

## Generating private keys

The private key is the actual identifier of a wallet. You can derive a private
key from a seed phrase with:

`./cardano-address key from-recovery-phrase <STYLE>`

Where <STYLE> is one of:

 * Byron (deprecated): used for Byron wallets created by Daedalus 
 * Icarus (deprecated): used for Byron wallets created in Yoroi
 * Jormungandr (deprecated): used for the incentivized testnet (ITN) 
 * Shelley: basically this is what you should use for new wallets 100% of the time

To create the private key from a seed phrase, execute:
```shell script
cat seed.txt | ./cardano-address key from-recovery-phrase Shelley > private_key.txt
cat private_key.txt
>xprv13zxmjna3a2v32wtnj60jw53fm4fy54737r9gj4pqape8sn4vgewcm8xsfeuh8f2404fung9lkgedhayalfwepa2he3qwmxppqv6tv36tc6epzmznfgy5j4glkg8lmahyr40lvhthzw9muumdf9guzrvsuqu7740q
```

The above inputs the seed phrase in `seed.txt` to command  
`cardano-address key from-recovery-phrase Shelley`, and outputs a 
`private_key.txt` file with your private key *in plain text*.

> **NOTE:** keeping seed phrases or private keys in plain text files is an 
> obvious security risk

## Generating a public key

The public key is the key you can use to identify your wallet on the blockchain.
You can obtain the public key for your wallet using:

```shell script
cat private_key.txt | ./cardano-address key public > public_key.txt
cat public_key.txt
>xpub1lxy8vqgzt00dq34u8476ulp60px6aggunztnf98vdn2c4rguwtjyh34jz9k9xjsff923lvs0lhmwg82l7ewhwyutheek6j23cyxepcqhau52r
```

Notice you can't send ADA to this address. For that you need an account.

### Understanding the HD wallet address format (BIP-44)

Basically from the private key, we derive more keys other than just the public
key: you can also derive accounts and payment addresses for each account. 

Everything is determined by a *derivation path*, in the following format:

```
private_key / purpose / coin_type / account_index / change / address_index
```

Some of these elements are pretty much fixed constants:

 * **purpose** = `1852` don't ask me why, it's a constant determined by a higher being
 
 * **coin_type** = `1815` which means Cardano's ADA 
 ([there's a standard for that](https://github.com/satoshilabs/slips/blob/master/slip-0044.md), 
 and we were assigned this number)
 
 * **change** = `0` which represents a "receiving" address. 1 means "change" address
  and as far as I know you won't usually need to worry about this.
  
If you are interested in generating payment addresses for a wallet, you simply
don't have to think about this stuff.
 
So let's move on to what was left unexplained:

 * **account_index**: typically `0`, but can be any number up to `2^31`. Accounts
 mean you can segregate payments received into your wallet into separate buckets.
 You could create a "personal account" on index `0`, a "business account" on 
 account index `1`, a "savings account" in index `100` and so on.
 
> **IMPORTANT:** All desktop/mobile wallets around currently support account 0 
> only. If you create account 1, 2 or whatever the transactions associated with
> these accounts will not show up 

 * **address_index**: a sequential index for a payment address, starting at `0`.
  Increment the address_index every time you need a new address to share with
  someone, so they can send funds to you. You can reuse payment addresses but 
  then multiple people will be able to see how much you are receiving through
  that same payment address. You can generate up to `2^31` addresses for each
  account.
  
> **IMPORTANT:** The BIP-44 protocol basically looks for 20 transactions after 
> the last transaction received. So if you receive a payment on address generated
> with address_index = 6, your wallet will only display the money received on 
> addresses from 0 to 26. 
>
> If you received payment on address with address_index = 30, the funds will not
> be displayed to you even though it's on the blockchain. It will only appear
> once there is a transaction in some address where address_index is between 10
> and 29.
>
> This applies for all current desktop/mobile/hardware wallets, so be aware.
> The gap limit can be customized on some wallets, but increasing it reduces 
> synchronization performance.


## Generating a public root key

To generate payment addresses for an account of your wallet you need the 
account derivation path - also known as the public root key. This is
essentially the initial section of the HD address:

```
private_key / purpose / coin_type / account_index 
``` 

Back to the command line, we can create the root key for account `0` with:

```shell script
cat private_key.txt | ./cardano-address key child 1852H/1815H/0H > account_0_key_path.txt
cat account_0_key_path.txt 
> xprv1fpwz27sk7yht78d4rwaq3a2e9p744w8xz0fe9rqpgkkxsk4vgew5tz6zfkw46xql705c8nzwulswap4zhq3vs3vwcj5cmtsqlxu3nstxheqxwqg2fj608vge2aja5yf5ug2603vkkp8s8jljktkrqzrqn5gykw0q
```

Here we input the private key, and generate a derivation path with the `1852`
and `1815` constants presented earlier, plus the account index. 

Notice that every constant in the path is followed by `H`, which stands for "hardened".
This means the key path was derived using the private key.

> **NOTICE:** although the `cardano-address` utility will give you an output that
> looks OK if you don't type in the 'H' after each constant, no payment address
> generated later on will be valid. If you send funds to such addresses they will
> be lost.

From the key path, we can generate the public root key:

```shell script
cat account_0_key_path.txt | ./cardano-address key public > account_0_public_root_key.txt 
cat account_0_public_root_key.txt
xpub1exevcxkw9ldsm7szljjyymesml0vzpc59swplz7ftnclk2yamm7kd0jqvuqs5n957wc3j4m9mggnfcs45lzedvz0q09l9vhvxqyxp8gpdvt5a
```

The public root key allows you or others to generate payment addresses securely
without having access to your private key. You can share this key with the world.

The public root key is useful to enable third parties to generate payment 
addresses in your behalf. Notice that the payment addresses generated from it
are bound to the account. In the example above, we can only derive payment 
addresses for account `0`.
  
## Generating the full derivation path

With the public root key ready, we only have to fill out the remaining bits
of the derivation path to get a full path:

`change / address_index`

As already mentioned, `change = 0`, and the address index starts from `0`. So
we can generate the full derivation address (still not a payment address) with:

```shell script
cat account_0_public_root_key.txt | ./cardano-address key child 0/0 > key_for_account_0_address_0.txt
cat key_for_account_0_address_0.txt 
> xpub1ccntmvkwgwlzv5r03vpy2n9nruy5x5d5kzww3egpvga7zf2v0gg2n7rlgrh5qnm7sja0wymccdydrvhnk6zgwy4llfcq8j0tzxexwmsn09nwm
```

That will generate a key which maps to account `0`, address `0` in your wallet. 
Notice the private key has not been used here.

We can keep generating full derivation paths for more addresses by incrementing
the `address_index` part:

```shell script
cat account_0_public_root_key.txt | ./cardano-address key child 0/1 > key_for_account_0_address_1.txt
cat key_for_account_0_address_1.txt 
xpub1kg7vl5c7nw9ee062u56xrv6dvpsd6lf3auwhggzh2e7edue4hdp6fntl6ux4w4wgy5gfh4tdmc2m6jszfzdnpcfyhhu72fjztcpsmms22e64k
```

> **NOTICE:** nothing prevents you from creating an address straight to some crazy
> index such as 1 million, e.g. `cardano-address key child 0/1000000`, but if 
> you send funds to a payment address generated from that key, it won't show up
> in your wallet as by default it expects one transaction every 20 addresses in 
> sequence.

## Generating a payment address

Finally, once you have the address key with the full derivation path, you can 
generate a payment address which people can send funds to:

```shell script
cat key_for_account_0_address_0.txt | ./cardano-address address payment --network-tag 1 > pay_to_account_0_address_0.txt
cat pay_to_account_0_address_0.txt
> addr1v8fet8gavr6elqt6q50skkjf025zthqu6vr56l5k39sp9aqlvz2g4
``` 

If you send `addr1v8fet8gavr6elqt6q50skkjf025zthqu6vr56l5k39sp9aqlvz2g4` to 
others, they will be able to send fund to you.

Notice that `--network-tag` was set to `1`. This parameter accepts two values:

 * `0` for testnet
 
 * `1` for mainnet
 
Make sure to use the appropriate network tag for your environment.

# Summary

The `cardano-address` utility can be integrated into your software to enable
the generation of seed phrases, public root keys for one or more accounts, and 
payment addresses that will work with the blockchain. It is by no means a 
wallet and doesn't intend to be used as such. 

Make sure to execute all steps in sequence, and test payments made to addresses
generated by your program on the testnet first (using `--network-tag 0`).

You will have to use a programming language the allows invoking the 
`cardano-address` executable from as a system call. I did this using Java and
if you are familiar with that programming language, you can look at the source 
code [here](https://github.com/uniVocity/free-commerce/blob/master/src/main/java/com/univocity/freecommerce/wallet/AddressManager.java),
with unit tests that perform the actions described in the tutorial
[here](https://github.com/uniVocity/free-commerce/blob/master/src/test/java/com/univocity/freecommerce/wallet/AddressManagerTest.java).

I will produce a library for other Java developers based on this source code
eventually. For now, feel free to copy these classes and use in your program. 

I Hope the information here is useful to you and helps you to get started writing
your own software for cardano!