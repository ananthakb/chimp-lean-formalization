# Chimp in Lean 4

Chimp is a lossless XOR-based compression codec for floating-point time series. It XORs each value's bit pattern with the previous one and encodes the difference using compact control tags and zero-bit information.

The goal is to implement the 64-bit Chimp encoder and decoder in Lean 4 and prove bit-for-bit round-trip correctness for every input sequence, preserving all floating-point bit patterns, including signed zeros and NaN payloads.

See [BLUEPRINT.md](BLUEPRINT.md) for the implementation and proof outline. Start with plain Chimp; Chimp128 and byte-for-byte compatibility with the Java implementation are possible follow-up work. This repository currently contains only these planning documents.

References:

- [Chimp paper](https://www.vldb.org/pvldb/vol15/p3058-liakos.pdf)
- [Authors' Java implementation](https://github.com/panagiotisl/chimp)
