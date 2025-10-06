# Huffman Encoding

## Overview
Huffman encoding is a lossless data compression algorithm that uses variable-length codes to represent characters based on their frequency of occurrence. More frequent characters get shorter codes, while less frequent characters get longer codes.

## Key Concepts

### Variable-Length Coding
- **Principle**: Assign shorter binary codes to more frequent characters
- **Optimality**: Huffman coding produces the minimum average code length for a given frequency distribution
- **Prefix Property**: No code is a prefix of another code, ensuring unambiguous decoding

### Entropy and Compression
```
Entropy H(X) = -Σ P(xi) × log₂(P(xi))
```
Where P(xi) is the probability of character xi occurring.

## Algorithm Details

### Step-by-Step Process

1. **Frequency Analysis**
   ```python
   def get_frequencies(text):
       freq = {}
       for char in text:
           freq[char] = freq.get(char, 0) + 1
       return freq
   ```

2. **Priority Queue Construction**
   ```python
   import heapq
   
   def build_priority_queue(frequencies):
       heap = []
       for char, freq in frequencies.items():
           node = HuffmanNode(char, freq)
           heapq.heappush(heap, node)
       return heap
   ```

3. **Huffman Tree Construction**
   ```python
   def build_huffman_tree(heap):
       while len(heap) > 1:
           left = heapq.heappop(heap)
           right = heapq.heappop(heap)
           
           merged = HuffmanNode(
               freq=left.freq + right.freq,
               left=left,
               right=right
           )
           heapq.heappush(heap, merged)
       
       return heapq.heappop(heap)
   ```

4. **Code Generation**
   ```python
   def generate_codes(root, code="", codes=None):
       if codes is None:
           codes = {}
       
       if root.char is not None:
           codes[root.char] = code if code else "0"
       else:
           generate_codes(root.left, code + "0", codes)
           generate_codes(root.right, code + "1", codes)
       
       return codes
   ```

## Complete Implementation

```python
import heapq
from collections import defaultdict, Counter

class HuffmanNode:
    def __init__(self, char=None, freq=0, left=None, right=None):
        self.char = char
        self.freq = freq
        self.left = left
        self.right = right
    
    def __lt__(self, other):
        return self.freq < other.freq
    
    def is_leaf(self):
        return self.left is None and self.right is None

class HuffmanEncoder:
    def __init__(self):
        self.tree = None
        self.codes = {}
    
    def build_tree(self, text):
        """Build Huffman tree from input text."""
        if not text:
            return None
        
        # Count character frequencies
        frequencies = Counter(text)
        
        # Handle single character case
        if len(frequencies) == 1:
            char = list(frequencies.keys())[0]
            self.tree = HuffmanNode(char, frequencies[char])
            self.codes = {char: "0"}
            return self.tree
        
        # Build priority queue
        heap = []
        for char, freq in frequencies.items():
            node = HuffmanNode(char, freq)
            heapq.heappush(heap, node)
        
        # Build tree
        while len(heap) > 1:
            left = heapq.heappop(heap)
            right = heapq.heappop(heap)
            
            merged = HuffmanNode(
                freq=left.freq + right.freq,
                left=left,
                right=right
            )
            heapq.heappush(heap, merged)
        
        self.tree = heapq.heappop(heap)
        return self.tree
    
    def generate_codes(self, node=None, code=""):
        """Generate Huffman codes by traversing the tree."""
        if node is None:
            node = self.tree
        
        if node.is_leaf():
            self.codes[node.char] = code if code else "0"
        else:
            self.generate_codes(node.left, code + "0")
            self.generate_codes(node.right, code + "1")
        
        return self.codes
    
    def encode(self, text):
        """Encode text using Huffman codes."""
        if not text:
            return "", {}, None
        
        self.build_tree(text)
        self.generate_codes()
        
        encoded = ""
        for char in text:
            encoded += self.codes[char]
        
        return encoded, self.codes, self.tree
    
    def decode(self, encoded_text, tree=None):
        """Decode Huffman-encoded text."""
        if tree is None:
            tree = self.tree
        
        if not encoded_text or tree is None:
            return ""
        
        decoded = ""
        current_node = tree
        
        for bit in encoded_text:
            if bit == "0":
                current_node = current_node.left
            else:
                current_node = current_node.right
            
            if current_node.is_leaf():
                decoded += current_node.char
                current_node = tree
        
        return decoded
    
    def get_compression_stats(self, original_text, encoded_text):
        """Calculate compression statistics."""
        original_bits = len(original_text) * 8
        compressed_bits = len(encoded_text)
        compression_ratio = original_bits / compressed_bits
        space_saved = (1 - compressed_bits / original_bits) * 100
        
        return {
            'original_size': len(original_text),
            'original_bits': original_bits,
            'compressed_bits': compressed_bits,
            'compression_ratio': compression_ratio,
            'space_saved_percent': space_saved,
            'average_code_length': compressed_bits / len(original_text)
        }

# Example usage and testing
def demo_huffman_encoding():
    """Demonstrate Huffman encoding with examples."""
    
    # Example 1: Simple text
    text1 = "hello world"
    encoder = HuffmanEncoder()
    encoded, codes, tree = encoder.encode(text1)
    decoded = encoder.decode(encoded, tree)
    stats = encoder.get_compression_stats(text1, encoded)
    
    print("=== Example 1: 'hello world' ===")
    print(f"Original: {text1}")
    print(f"Encoded: {encoded}")
    print(f"Codes: {codes}")
    print(f"Decoded: {decoded}")
    print(f"Compression ratio: {stats['compression_ratio']:.2f}:1")
    print(f"Space saved: {stats['space_saved_percent']:.1f}%")
    print()
    
    # Example 2: Highly repetitive text
    text2 = "aaaaabbbbcccdde"
    encoded2, codes2, tree2 = encoder.encode(text2)
    decoded2 = encoder.decode(encoded2, tree2)
    stats2 = encoder.get_compression_stats(text2, encoded2)
    
    print("=== Example 2: Highly repetitive text ===")
    print(f"Original: {text2}")
    print(f"Encoded: {encoded2}")
    print(f"Codes: {codes2}")
    print(f"Decoded: {decoded2}")
    print(f"Compression ratio: {stats2['compression_ratio']:.2f}:1")
    print(f"Space saved: {stats2['space_saved_percent']:.1f}%")
    print()
    
    # Example 3: Single character
    text3 = "aaaaaaaaaa"
    encoded3, codes3, tree3 = encoder.encode(text3)
    decoded3 = encoder.decode(encoded3, tree3)
    stats3 = encoder.get_compression_stats(text3, encoded3)
    
    print("=== Example 3: Single character repetition ===")
    print(f"Original: {text3}")
    print(f"Encoded: {encoded3}")
    print(f"Codes: {codes3}")
    print(f"Decoded: {decoded3}")
    print(f"Compression ratio: {stats3['compression_ratio']:.2f}:1")
    print(f"Space saved: {stats3['space_saved_percent']:.1f}%")

if __name__ == "__main__":
    demo_huffman_encoding()
```

## Huffman Tree Visualization

### Tree Structure Example
For text "hello world":

```
Frequencies: {'h':1, 'e':1, 'l':3, 'o':2, ' ':1, 'w':1, 'r':1, 'd':1}

Tree:
                    Root (11)
                   /         \
               Internal(4)    'l'(3)
              /         \
         Internal(2)    Internal(2)
        /         \    /         \
      'o'(2)   Internal(2)  ' '(1)  Internal(1)
              /         \         /         \
           'h'(1)     'e'(1)   'w'(1)   Internal(1)
                                           /         \
                                        'r'(1)     'd'(1)

Codes:
'l': 0
'o': 10
'h': 1100
'e': 1101
' ': 1110
'w': 11110
'r': 111110
'd': 111111
```

## Advanced Features

### Canonical Huffman Coding
```python
def canonical_huffman_encode(frequencies):
    """Generate canonical Huffman codes for better efficiency."""
    # Sort characters by frequency, then alphabetically
    sorted_chars = sorted(frequencies.items(), key=lambda x: (x[1], x[0]))
    
    # Build codes maintaining canonical property
    codes = {}
    current_code = 0
    current_length = 1
    
    for char, freq in sorted_chars:
        codes[char] = format(current_code, f'0{current_length}b')
        current_code += 1
        
        # Check if we need to increase code length
        if current_code >= (1 << current_length):
            current_code = 0
            current_length += 1
    
    return codes
```

### Adaptive Huffman Coding
```python
class AdaptiveHuffmanEncoder:
    """Adaptive Huffman encoder that updates frequencies during encoding."""
    
    def __init__(self):
        self.frequencies = {}
        self.tree = None
        self.codes = {}
    
    def update_frequency(self, char):
        """Update character frequency and rebuild tree."""
        self.frequencies[char] = self.frequencies.get(char, 0) + 1
        self.tree = self.build_tree_from_frequencies()
        self.generate_codes()
    
    def encode_adaptive(self, text):
        """Encode text with adaptive frequency updates."""
        encoded = ""
        
        for char in text:
            if char not in self.codes:
                # New character - send escape code + character
                encoded += self.codes.get('ESC', '') + format(ord(char), '08b')
            
            encoded += self.codes[char]
            self.update_frequency(char)
        
        return encoded
```

## Performance Analysis

### Time Complexity
- **Tree Construction**: O(n log n) where n is the number of unique characters
- **Encoding**: O(m) where m is the length of input text
- **Decoding**: O(m) where m is the length of encoded text

### Space Complexity
- **Tree Storage**: O(n) where n is the number of unique characters
- **Code Table**: O(n) for storing character codes

### Compression Efficiency
```python
def calculate_theoretical_compression(text):
    """Calculate theoretical compression limits."""
    frequencies = Counter(text)
    total_chars = len(text)
    
    # Calculate entropy
    entropy = 0
    for char, count in frequencies.items():
        probability = count / total_chars
        entropy -= probability * math.log2(probability)
    
    # Theoretical minimum bits
    theoretical_bits = total_chars * entropy
    
    return {
        'entropy': entropy,
        'theoretical_min_bits': theoretical_bits,
        'original_bits': total_chars * 8
    }
```

## Applications

### File Compression
- **ZIP files**: Combined with LZ77 algorithm
- **GZIP**: HTTP compression, log files
- **BZIP2**: Combined with Burrows-Wheeler Transform

### Image Compression
- **JPEG**: Used in entropy coding stage
- **PNG**: Combined with LZ77 (DEFLATE)

### Network Protocols
- **HTTP/2**: Header compression using HPACK
- **Wireless protocols**: Data compression for bandwidth efficiency

## Limitations and Considerations

### Disadvantages
1. **Two-pass algorithm**: Requires frequency analysis before encoding
2. **Tree overhead**: Must store tree structure with compressed data
3. **Poor for small files**: Overhead may exceed compression benefits
4. **Not optimal for all distributions**: Fixed Huffman codes may not be optimal

### When to Use
- **Text with known frequency distribution**
- **Large files where tree overhead is negligible**
- **Applications where decompression speed is important**
- **Memory-constrained environments**

### Alternatives
- **Arithmetic coding**: Better compression but more complex
- **LZ77/LZ78**: Better for general-purpose compression
- **Adaptive algorithms**: Better for unknown frequency distributions

## Optimization Techniques

### Tree Optimization
```python
def optimize_tree_structure(tree):
    """Optimize tree structure for better cache performance."""
    # Implement tree balancing or reordering
    pass
```

### Memory Management
```python
class MemoryEfficientHuffman:
    """Memory-efficient Huffman implementation for large datasets."""
    
    def __init__(self, max_memory_mb=100):
        self.max_memory = max_memory_mb * 1024 * 1024
        self.current_memory = 0
    
    def streaming_encode(self, input_file, output_file):
        """Encode large files in streaming fashion."""
        # Implement streaming compression
        pass
```

This comprehensive guide covers Huffman encoding from basic concepts to advanced implementations, providing both theoretical understanding and practical code examples.
