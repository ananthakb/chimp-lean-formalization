# Blueprint

Implement plain 64-bit Chimp in Lean 4, then prove bit-for-bit round-trip correctness.

1. **Specify the format.** Model values as `BitVec 64`, with no floating-point arithmetic. Write the first word verbatim. For later words, let `d` be the XOR with the previous word, `t` its trailing-zero count, and `l` its leading-zero count rounded down to a bucket in `{0, 8, 12, 16, 18, 20, 22, 24}`. Encode bucket indices in three bits. Specify the four two-bit control tags, in this priority order:

   | Tag | Case and payload |
   | --- | --- |
   | `00` | `d = 0`: repeat the previous word; no payload. |
   | `01` | Nonzero `d`, `t > 6`: bucket index, six-bit significant width `64 - l - t`, and the significant XOR bits. |
   | `10` | Otherwise, `l` equals the stored leading-zero count: the low `64 - l` XOR bits. |
   | `11` | Otherwise: bucket index and the low `64 - l` XOR bits. |

2. **Prove bitstream primitives.** Define fixed-width reads and writes with a fixed bit order and explicit errors for insufficient input. For every width `k`, field `v : BitVec k`, and suffix `s`, prove that reading `writeBits v ++ s` returns `v` and exactly `s`. Use this suffix-preservation lemma to compose fields.

3. **Implement one-value steps.** Track the previous word and an optional stored leading-zero count. Handle initialization separately; for later words emit or parse the tag and payload, reconstruct `d`, and XOR it with the previous word. Invalidate the stored count after `00` or `01`, retain it after `10`, and set it after `11`. Update the previous word after every step.

4. **Prove step correctness.** Establish state validity and bucket/width bounds. By cases on initialization and the four tags, prove that decoding one freshly encoded word recovers that word, preserves any following stream suffix, and leaves encoder and decoder states matching.

5. **Lift to sequences.** Define list encoding and count-directed decoding from matching initial states. Generalize the induction hypothesis over valid states and stream suffixes, then induct on the input list to prove `decode xs.length (encode xs) = .ok xs`, including the empty list. An explicit count avoids reserving any word as an end marker.

## Theorems to prove, informally

These are proof goals for the future implementation.

1. **Field round trip.** For every width, every bit pattern of that width, and every following stream, reading the freshly written field returns exactly that pattern and leaves the following stream untouched.

2. **Bucket and payload correctness.** For every nonzero XOR difference, rounding down its leading-zero count discards only zeros, and decoding its three-bit bucket index recovers the chosen bucket. Every emitted field fits its declared width. Restoring the omitted zeros to each tag's payload reconstructs the entire XOR difference exactly.

3. **XOR cancellation.** For any previous word `p` and current word `w`, XORing `p` with the difference `p XOR w` recovers `w`. A zero difference recovers `p` itself.

4. **State invariants.** The initial state is valid. Encoding any word from a valid state produces a valid next state; decoding the emitted bits from a matching state preserves validity. The previous word and optional stored leading-zero count satisfy the specified initialization and tag-update rules.

5. **One-word round trip.** For any word and valid matching encoder/decoder states, decoding that word's encoding succeeds, returns exactly the word, and leaves the states matching. It consumes precisely the emitted bits and preserves any following stream. Prove this for the first word and separately for all four tags.

6. **Sequence round trip with a suffix.** From any valid matching states, encoding a finite list and appending any stream, then decoding exactly the list's length, returns that list, leaves precisely the appended stream, and produces matching final states. Prove this by induction using the one-word theorem.

7. **End-to-end correctness.** Starting from the initial states, for every finite list `xs` of 64-bit words, `decode xs.length (encode xs) = .ok xs`. This includes the empty list and all bit patterns, including signed zeros, infinities, and every NaN payload.


