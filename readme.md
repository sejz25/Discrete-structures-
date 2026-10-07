# Finite Automata in Practice: From Compiler Lexers to the TCP Handshake

Finite automata look like pure theory in a Discrete Structures or Compiler Design course. In reality they sit inside compilers, firewalls and network stacks. This article explains the concept, then shows two applications: lexical analysis in compilers and connection tracking in TCP.

## What Is a Finite Automaton?

A deterministic finite automaton (DFA) is defined by five components:

| Symbol | Meaning |
|---|---|
| Q | finite set of states |
| Σ | input alphabet |
| δ | transition function: (state, input) → next state |
| q₀ | start state |
| F | accepting states |

The machine reads input one symbol at a time and moves between states according to δ. It remembers nothing except its current state. That limitation is also its strength: a DFA is simple to verify and fast to run, since each symbol costs one table lookup.

## Application 1: Lexical Analysis in Compilers

The first phase of a compiler, the lexer, converts source code into tokens such as identifiers, numbers and keywords. Each token type is described by a regular expression. For example, an identifier in many languages is:

```text
[a-zA-Z_][a-zA-Z0-9_]*
```

Tools such as Lex and Flex convert these expressions into an NFA, then into a DFA, and generate a scanner from it. An NFA may have several possible moves for one input, which makes it easy to build from a regular expression. The subset construction then turns it into a DFA with exactly one move per input, which is what makes the generated scanner fast. The DFA for identifiers is tiny:

<img width="754" height="385" alt="image" src="https://github.com/user-attachments/assets/07556c27-4f2b-40b1-998d-eb602fab4853" />


Tracing `count_1`: the first letter `c` moves q0 to q1, and every later letter, digit or underscore keeps the machine in q1. The input ends in the accepting state, so it is a valid identifier. The input `1count` has no transition from q0 on a digit, so it is rejected. This is why regular expressions and finite automata are called equivalent: they describe exactly the same class of languages.

## Application 2: The TCP Handshake as a State Machine

Network protocols are state machines too. The diagram shows a simplified server side of a TCP connection.

<img width="1411" height="358" alt="image" src="https://github.com/user-attachments/assets/8d1e4ca6-b8f8-4d6a-bf5c-d4d029ece7d7" />


The states are LISTEN, SYN_RECEIVED, ESTABLISHED and CLOSED, and the inputs are SYN, ACK, FIN and RST. In code, δ is a dictionary:

```python
DELTA = {
    ("LISTEN", "SYN"): "SYN_RECEIVED",
    ("SYN_RECEIVED", "ACK"): "ESTABLISHED",
    ("SYN_RECEIVED", "RST"): "LISTEN",
    ("ESTABLISHED", "FIN"): "CLOSED",
    ("ESTABLISHED", "RST"): "CLOSED",
}
```

Any pair missing from the table is rejected. Output from running [`tcp_fsm.py`](tcp_fsm.py):

```text
normal handshake + close
  LISTEN        --SYN--> SYN_RECEIVED
  SYN_RECEIVED  --ACK--> ESTABLISHED
  ESTABLISHED   --FIN--> CLOSED

ACK with no SYN first
  LISTEN        --ACK--> REJECT (no such transition)

half-open tracking across sources
  sources tracked: 8, half-open: 6, established: 2
  half-open > 5? ALERT
```

This model is simplified. Real TCP has more states, and the server's SYN-ACK reply is an output, so it does not appear as an input. RFC 793 gives the full diagram.

## Why This Matters for Security

- **Stateful firewalls** track each connection's state and drop packets that do not match a legal transition, like the unexpected ACK above.
- **SYN flood detection:** an attacker sends many SYNs and never completes the handshake, leaving many connections stuck in SYN_RECEIVED. Counting half-open connections per source, as in the last block of the output, is a simple detection idea. The threshold of 5 is only for the demo.
- **Protocol parsers** are often written as state machines. Gaps in their transitions can be exploited with malformed input.

## Other Real-Life Applications

- **Text search and pattern matching:** tools such as grep and many regex engines compile a pattern into an automaton, so a file can be scanned in a single pass.
- **Input validation:** formats like phone numbers, dates and email-style patterns are commonly checked with regular expressions, which are finite automata in disguise.
- **Embedded and control systems:** traffic lights, vending machines and elevator controllers are classic state machines. Each state has a clear set of legal next steps, which makes the behaviour easy to test.
- **Software design:** game characters, user-interface flows and order-processing workflows are often modelled as states and transitions to avoid impossible situations.

## Importance in Computer Science

Finite automata form the base of the Chomsky hierarchy and connect several areas: compiler lexers, text search, network protocols, hardware control logic and security tooling. They also show their own limits. An FSM cannot count without bound, which is why my half-open counter lives outside the machine. Matching nested structures such as balanced parentheses needs pushdown automata, the next step in parsing theory.

## Conclusion

Finite automata are one of the few ideas that appear unchanged in a compiler, a firewall and a vending machine. Learning them well gives a practical way to read regular expressions, to design protocols and tools with clear behaviour, and to describe attacks in terms of states rather than vague symptoms. For anyone moving from theory into compiler design or network security, they are the right first model to master.

## Video

[![Finite State Machine (Finite Automata)](https://img.youtube.com/vi/Qa6csfkK7_I/0.jpg)](https://www.youtube.com/watch?v=Qa6csfkK7_I)

## References

- Saylor University, CS202: Discrete Structures: https://learn.saylor.org/course/cs202
- RFC 793, Transmission Control Protocol: https://www.rfc-editor.org/rfc/rfc793
