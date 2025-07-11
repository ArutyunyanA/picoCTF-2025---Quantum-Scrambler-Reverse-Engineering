# picoCTF-2025---Quantum-Scrambler-Reverse-Engineering
picoCTF Chalenges - Quantum Scrambler Reverse Engineering

## picoCTF Chalenges - Quantum Scrambler Reverse Engineering

### Step - 1 Analyzing the Source Code

The provided source code defines a custom scrambling function that obfuscates the flag by mutating a list of characters represented in hex.

#### Function Breakdown:

- `get_flag()`:
  - Reads the flag from `flag.txt`.
  - Converts each character to its hexadecimal representation (`hex(ord(c))`) and wraps it inside a list.

- `scramble(L)`:
  - Iteratively modifies the list `L`:
    - At each iteration, the element at index `i-2` is concatenated with the element at `i-1` (i.e., `A[i-2] += A.pop(i-1)`).
    - The element at index `i-1` is then modified to include a nested copy of the list up to `i-2` (i.e., `A[i-1].append(A[:i-2])`).
  - This results in a nested and scrambled structure that still contains all the original hex strings.

The structure returned by `scramble()` is printed and needs to be captured for analysis. It is a deeply nested list of strings representing hex characters. These must be flattened and converted back to characters to reveal the flag.

---

### Step - 2 Get Encrypted Data

Connect to the remote server to obtain the encrypted data:

```bash
nc verbal-sleep.picoctf.net 63518
```

Save the output (nested array of hex strings) into a file named `arrays.py`. The file should define a variable `array` like so:

```python
array = [...]['0x70', '0x69', '0x63', [], '0x6f', '0x43', [['0x70', '0x69']], '0x54'...]  # full list of strings as printed by the remote service
```

---

### Step - 3 Writing Algorithm to Decrypt Data

Create a script named `reverse_scrambler.py` with the following logic to flatten the nested structure and decode the hex values:

```python
from arrays import array

# Recursively unpacks nested lists containing hex strings into a flat list of strings
def unpack_array(data: list) -> list[str]:
    result = []
    for x in data:
        if isinstance(x, list):
            for s in x:
                if isinstance(s, str):
                    result.append(s)  # Append strings inside inner lists
        elif isinstance(x, str):
            result.append(x)  # Append top-level strings
    return result

# Converts a list of hex strings (e.g., "0x63") to the corresponding ASCII characters
def hex_to_string(hex_list: list) -> str:
    flag = "".join([chr(int(char, 16)) for char in hex_list])
    return flag

# Main function to unpack the array, decode hex values, and print the final flag
def main():
    data = array  # Load the nested encrypted data from arrays.py
    hex_data = unpack_array(data)  # Flatten the structure
    flag = hex_to_string(hex_data)  # Decode to ASCII string
    print(f"[+] Found the flag: {flag}")

# Entry point of the script
if __name__ == "__main__":
    main()

```

---

### Output

```bash
$ python3 reverse_scrambler.py 
[+] Found the flag: picoCTF{python_is_weirdef8ea0cf}
```

