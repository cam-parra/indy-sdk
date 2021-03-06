# Send a Secure Message
Indy-SDK Developer Walkthrough #3, Rust Edition

[ [Java](../../not-yet-written.md) | [.NET](../../not-yet-written.md) | [Node.js](../../not-yet-written.md) | [Objective C](../../not-yet-written.md)  | [Python](../python/README.md) ]


## Prerequisites

Setup your workstation and indy development virtual machine. See [prerequisites](../../prerequisites.md).

## Steps

### Step 1

In your normal workstation OS (not the VM), open a rust editor of your
choice and paste the code from [template.rs](template.rs)
into a new doc. We will be modifying this code in later steps.

Save the doc as `send-secure-msg.rs`

This is a very simple app that allows you to act as a sender or a receiver
of a message. Take a minute to understand its basic structure: it runs a
loop, asking you for input.

It recognizes three commands: `read`, `prep`, and `quit`.

### Step 2

Run the app on your workstation.
You should see some simple text output. Try a few commands.

Type `quit` when you're done.

### Step 3

Now we get to begin adding interesting features.

The first thing we need to do is give the app the DIDs and the keys it
needs to communicate. Open [step3.rs](step3.rs) in a text editor and copy
its contents into `send-secure-msg.rs`, next replace the stub of
the `init()` function on `let (wallet_handle, verkey, other_verkey) = init();`.

Save the updated version of `send-secure-msg.rs`. Now take a minute and study the changes.

First `init()` asks the user for their name. It then invokes indy-sdk's
`create_wallet()` function to make a wallet associated with a fictional
pool (ledger). It records the wallet handle. Once the wallet has been
created, it opens the wallet and asks indy-sdk to generate a DID and a
public verkey + private signing key pair, storing all that information
in the wallet.

Finally, `init()` asks the user to provide the DID and verkey of the
other party that will be exchanging messages, and it returns a tuple of
all the information it's accumulated, except for the secret signing key
that remains in the wallet.

### Step 4

Run the app again.

You should be prompted for a name. Say "Alice". Then the app should show you the DID and verkey it generated, and ask you for the DID and verkey of the other party.

Press **CTRL+C** to kill the app.

```
cargo run
Who are you? Alice
My DID and Verkey: ChST7uE2KH3hscqycs5mVf 7NmnhUTnqqh1xydTSZNZCp1wTt3HAua7gA5T2odzLSTf
Other party's DID and Verkey? ^C
```

We will eventually run this app twice--in one window as Alice, and in another
window as Bob. Each instance of the app will generate its own keys and
you can use copy/paste to share them with the other window.

### Step 5

Now we need to add secure encryption using indy crypto's [`auth_crypt()`](https://github.com/hyperledger/indy-sdk/blob/master/libindy/src/api/crypto.rs#L272)
primitive.

Copy the contents of [step5.rs](step5.rs) into `send-secure-msg.rs` and replace the stub of
the `prep()` function on `prep(wallet_handle, &verkey, &other_verkey);`.

Save the updated version of `send-secure-msg.rs`. Now take a minute and study the changes.

The `prep()` function is designed to be called with a string argument entered
on the command line.

For example, when the user types `prep Hello, world`, the `msg` parameter of `prep()` receives the value `"Hello, world"`. Prep saves this message into a text file (so you can inspect it--not because it needs to do any file I/O). It then turns around and reads the bytes it has just written, and calls `auth_crypt()` to convert those bytes into a version that is encrypted especially for the private channel between two identities. The receiver will know that the sender created the message, that only the receiver can interpret it, and that it has not been tampered with.

If you like, try running the updated app in your development VM again. When
prompted for the DID and verkey of the other party, just paste values produced 
during a previous run (e.g., `ChST7uE2KH3hscqycs5mVf 7NmnhUTnqqh1xydTSZNZCp1wTt3HAua7gA5T2odzLSTf`) and press enter. Then try typing `prep Hello, world` and inspecting the `message.txt` that are created.

Press **CTRL+C** to kill the app.

### Step 6

The final feature that our app needs is the ability to read encrypted data.
Copy the contents of [step6.rs](step6.rs) into `send-secure-msg.rs` and replace the stub of the `read()`
function on `read(wallet_handle, &verkey);`.

Save the updated version of `send-secure-msg.rs`. Now take a minute and study the
changes.

The read function is very simple. It just copies the content of the
`message.txt` file from disk into memory and then calls indy crypto's
`auth_decrypt()` function on the byte array. The output is a tuple that
contains the verkey of the sender and the decrypted value if the
message was encrypted for the key of the recipient--or an exception if
not.

### Step 7

Now let's put this all together.

In your indy development VM, create two shells (command prompts). Both should have the location of your code as their current working directory. Start one copy of your scripts in the first window (`cargo run`), and another copy in the second window.

In the first window, when prompted for your name, say "Alice".  In the
second window, say "Bob".

In the Alice window, you should now see a line that specifies a DID and
verkey for Alice. It should look something like this:

```
My DID and Verkey:  ChST7uE2KH3hscqycs5mVf 7NmnhUTnqqh1xydTSZNZCp1wTt3HAua7gA5T2odzLSTf
```

Copy these two strings (everything after `"My DID and Verkey: "`) to your
clipboard.

Navigate to the Bob window. Bob's window should have a different did and verkey displayed, and should be prompting for Alice's info.

Paste Alice's information into Bob's window and press Enter.

Copy Bob's information and paste it into Alice's window.

**Note:** The process of copying and pasting between two windows is a simplistic way to model more sophisticated onboarding workflows in the Sovrin ecosystem. Within the Sovrin ecosystem parties receive a trusted mutual introduction, or where one scans a QR code from the other, or where they exchange information over the phone or face to face.

Both windows should now be displaying a prompt again.

In the Alice window, type `prep Hi, Bob.` and press **Enter**.

In the Bob window, type `read` and press **Enter**.

Bob's window should display a tuple that it decrypts from Alice; the first
element in the tuple is Alice's verkey (check its value to confirm); the
second is the text, `"Hi, Bob."`. Alice has sent Bob an encrypted message.