# Compression Algorithms

## Overview
Compression algorithms reduce the size of data by eliminating redundancy and using efficient encoding schemes. They are essential for storage optimization, network transmission, and data processing.

## Types of Compression

### 1. Lossless Compression
- **Definition**: No data is lost during compression and decompression
- **Use Cases**: Text files, databases, executable files
- **Examples**: ZIP, GZIP, PNG, FLAC

### 2. Lossy Compression
- **Definition**: Some data is permanently lost, but acceptable for the use case
- **Use Cases**: Images, audio, video
- **Examples**: JPEG, MP3, MP4

## Common Compression Algorithms

### Run-Length Encoding (RLE)
**Principle**: Replace consecutive identical data elements with a count and the element.

```
Original: AAAAAABBBCCCCDDDDDDD
Compressed: 6A3B4C7D
Compression Ratio: 21/8 = 2.625:1
```

**Implementation**:
```python
def run_length_encode(data):
    if not data:
        return ""
    
    encoded = ""
    current_char = data[0]
    count = 1
    
    for i in range(1, len(data)):
        if data[i] == current_char:
            count += 1
        else:
            encoded += f"{count}{current_char}"
            current_char = data[i]
            count = 1
    
    encoded += f"{count}{current_char}"
    return encoded

def run_length_decode(encoded):
    decoded = ""
    i = 0
    while i < len(encoded):
        count = int(encoded[i])
        char = encoded[i + 1]
        decoded += char * count
        i += 2
    return decoded
```

### Lempel-Ziv-Welch (LZW)
**Principle**: Build a dictionary of frequently occurring patterns during compression.

**Algorithm**:
1. Initialize dictionary with all possible single characters
2. Read input and find longest string in dictionary
3. Output the dictionary index
4. Add the string + next character to dictionary

**Example**:
```
Input: "ABABABABA"
Dictionary: {0: 'A', 1: 'B', 2: 'AB', 3: 'BA', 4: 'ABA', 5: 'BAB', 6: 'ABAB'}
Output: [0, 1, 2, 4, 6]
```

### Burrows-Wheeler Transform (BWT)
**Principle**: Reorganize data to group similar characters together, making it more compressible.

**Process**:
1. Create all rotations of the input string
2. Sort rotations lexicographically
3. Take the last column as the transformed string

**Example**:
```
Input: "BANANA"
Rotations and sorting:
ABANAN
ANABAN
ANANAB
BANANA
NABANA
NANABA

Sorted:
ABANAN
ANABAN
ANANAB
BANANA
NABANA
NANABA

BWT result: "NNBAAA"
```

## Huffman Coding

### Overview
Huffman coding is a lossless data compression algorithm that uses variable-length codes based on character frequencies.

### Algorithm Steps

1. **Frequency Analysis**: Count frequency of each character
2. **Build Priority Queue**: Create nodes for each character with their frequencies
3. **Build Huffman Tree**: 
   - Remove two lowest frequency nodes
   - Create parent node with combined frequency
   - Insert parent back into queue
   - Repeat until one node remains
4. **Generate Codes**: Traverse tree to assign binary codes
5. **Encode**: Replace characters with their Huffman codes

### Example Implementation

```python
import heapq
from collections import defaultdict

class HuffmanNode:
    def __init__(self, char=None, freq=0, left=None, right=None):
        self.char = char
        self.freq = freq
        self.left = left
        self.right = right
    
    def __lt__(self, other):
        return self.freq < other.freq

def build_huffman_tree(text):
    # Count frequencies
    freq = defaultdict(int)
    for char in text:
        freq[char] += 1
    
    # Create priority queue
    heap = []
    for char, count in freq.items():
        node = HuffmanNode(char, count)
        heapq.heappush(heap, node)
    
    # Build tree
    while len(heap) > 1:
        left = heapq.heappop(heap)
        right = heapq.heappop(heap)
        
        merged = HuffmanNode(freq=left.freq + right.freq, left=left, right=right)
        heapq.heappush(heap, merged)
    
    return heapq.heappop(heap)

def generate_codes(root, code="", codes={}):
    if root.char is not None:
        codes[root.char] = code if code else "0"
    else:
        generate_codes(root.left, code + "0", codes)
        generate_codes(root.right, code + "1", codes)
    return codes

def huffman_encode(text):
    tree = build_huffman_tree(text)
    codes = generate_codes(tree)
    
    encoded = ""
    for char in text:
        encoded += codes[char]
    
    return encoded, codes, tree

def huffman_decode(encoded, tree):
    decoded = ""
    current = tree
    
    for bit in encoded:
        if bit == "0":
            current = current.left
        else:
            current = current.right
        
        if current.char is not None:
            decoded += current.char
            current = tree
    
    return decoded
```

### Example Usage

```python
text = "hello world"
encoded, codes, tree = huffman_encode(text)

print(f"Original: {text}")
print(f"Encoded: {encoded}")
print(f"Codes: {codes}")
print(f"Compression ratio: {len(text) * 8} / {len(encoded)} = {len(text) * 8 / len(encoded):.2f}")

decoded = huffman_decode(encoded, tree)
print(f"Decoded: {decoded}")
```

### Huffman Tree Visualization

```
        Root (11)
       /         \
    'l'(3)     Internal(8)
              /         \
         Internal(5)    ' '(2)
        /         \
    'o'(2)      Internal(3)
              /         \
           'h'(1)      'e'(1)
```

## Compression Performance Metrics

### Compression Ratio
```
Compression Ratio = Original Size / Compressed Size
```

### Compression Efficiency
```
Efficiency = (1 - Compressed Size / Original Size) × 100%
```

### Time Complexity
- **Huffman Coding**: O(n log n) where n is the number of unique characters
- **RLE**: O(n) where n is the input length
- **LZW**: O(n) where n is the input length

## Applications

### File Compression
- **ZIP**: Uses LZ77 + Huffman coding
- **GZIP**: Uses LZ77 + Huffman coding + checksums
- **BZIP2**: Uses BWT + Move-to-front + Huffman coding

### Image Compression
- **PNG**: Uses LZ77 variant (DEFLATE)
- **JPEG**: Uses DCT + quantization + Huffman coding

### Audio Compression
- **MP3**: Uses psychoacoustic models + Huffman coding
- **FLAC**: Uses linear prediction + Rice coding

### Video Compression
- **H.264**: Uses motion compensation + DCT + entropy coding
- **H.265**: Advanced motion compensation + improved entropy coding

## Choosing the Right Algorithm

| Algorithm | Best For | Pros | Cons |
|-----------|----------|------|------|
| RLE | Simple repetitive data | Fast, simple | Poor for complex data |
| Huffman | Text with character frequency variations | Optimal for known frequencies | Requires two-pass encoding |
| LZW | General-purpose compression | Good compression ratio | Patent issues (expired) |
| BWT | Text with local patterns | Very good for text | Complex implementation |

## Implementation Tips

1. **Memory Management**: For large files, use streaming compression
2. **Dictionary Size**: Limit dictionary size in LZW to prevent memory issues
3. **Frequency Analysis**: Use sampling for very large datasets
4. **Error Handling**: Always include checksums for data integrity
5. **Performance**: Consider parallel processing for large files

## Future Trends

- **Machine Learning**: AI-based compression algorithms
- **Hardware Acceleration**: GPU-accelerated compression
- **Adaptive Algorithms**: Algorithms that learn optimal compression strategies
- **Quantum Compression**: Theoretical quantum compression algorithms
