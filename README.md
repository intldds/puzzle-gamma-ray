# puzzle-gamma-ray

**DO NOT FORK THE REPOSITORY, AS IT WILL MAKE YOUR SOLUTION PUBLIC. INSTEAD, CLONE IT AND ADD A NEW REMOTE TO A PRIVATE REPOSITORY, OR SUBMIT A GIST**

Trying it out
=============

Use `cargo run --release` to see it in action

Submitting a solution
=====================

[Submit a solution](https://xng1lsio92y.typeform.com/to/ca4f2sT0)

[Submit a write-up](https://xng1lsio92y.typeform.com/to/zH5sNzep)

Puzzle description
==================

    |___  /| | / / | | | |          | |
       / / | |/ /  | |_| | __ _  ___| | __
      / /  |    \  |  _  |/ _` |/ __| |/ /
    ./ /___| |\  \ | | | | (_| | (__|   <
    \_____/\_| \_/ \_| |_/\__,_|\___|_|\_\

Bob was deeply inspired by the Zcash design [1] for private transactions and had some pretty cool ideas on how to adapt it for his requirements. He was also inspired by the Mina design for the lightest blockchain and wanted to combine the two. In order to achieve that, Bob used the MNT6753 cycle of curves to enable efficient infinite recursion, and used elliptic curve public keys to authorize spends. He released a first version of the system to the world and Alice soon announced she was able to double spend by creating two different nullifiers for the same key... 

[1] https://zips.z.cash/protocol/protocol.pdf

Puzzle write-up
==================
The goal of this puzzle is to create a new nullifier (`nullifier_hack`) in order to be able to double spend using the protocol.

The reason why this might be possible is that Bob used the MNT7653 cycle of curves, which is represented in **Weierstrass form** in the circuit. 
In this cycle of curves, **both (x, y) and (x, -y) are valid points for spending**.

Therefore, only the *x-coordinate* of an elliptic curve point is used as the leaf node and there are **2 possible
points** with the same *x-coordinate*.

Considering that we already know `(secret * G)`, we need to find its negation, that is `(-secret * G)`.
The tricky part is that `leaked_secret` is specified in `MNT4BigFr`, but the *x-coordinate* is computed over `MNT6BigFr`,
so we need to calculate our `secret_hack` as `MNT4BigFr::from(MNT6BigFr::MODULUS) - leaked_secret`.

Once we calculate our `secret_hack`, all we have to do is calculate the `nullifier_hack` the same way as the original one 
was calculated: by using the **Poseidon hash function** on the *new* secret.

Now, our `secret_hack` and `nullifier_hack` are able to pass the circuit checking parameters.

This was a fun puzzle which allowed me to have a better understanding of **nullifiers** and increase my awareness to 
the cycle of curves that we use and their potential vulnerabilities.