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

Possible follow-up work: Chimp128 and byte-for-byte compatibility with the authors' Java implementation, including framing and padding.
