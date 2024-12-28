# lualzw
A relatively fast LZW compression algorithm in pure lua

# encoding and decoding
Lossless compression for any text. The more repetition in the text, the better.

16 bit encoding is used. So each 8 bit character is encoded as 16 bit.
This means that the dictionary size is 65280.

Any special characters like `äöå` that are represented with multiple characters are supported. The special characters are split up into single characters that are then encoded and decoded. 

While compressing, the algorithm checks if the result size gets over the input. If it does, then the input is not compressed and the algorithm returns the input prematurely as the compressed result.

The `zeros` branch contains a version that does not add additional null `\0` characters to the input when encoding. Any existing null characters in input string are preserved as nulls however so make sure your input does not contain nulls.

# usage
```lua
local lualzw = require("lualzw")

local input = "foofoofoofoofoofoofoofoofoo"
local compressed = assert(lualzw.compress(input))
local decompressed = assert(lualzw.decompress(compressed))
assert(input == decompressed)
```

# errors
Returns nil and an error message when the algorithm fails to compress or decompress.

# speed
Times are in seconds.
Both have the same generated input.
The values are an average of 10 tries.

Note that compressing random generated inputs results usually in bigger result than original. In these cases the algorithms do not compress and return input instead and thus compression result is 100% of input.

lualzw is at an advantage in cases where compression cannot be done as it stops prematurely and LibCompress does not.
Also lualzw is at an advantage in cases where compression can be done as it has a larger dictionary in use.

Input: 1000000 random generated bytes converted into string

algorithm|compress|decompress|result % of input
---------|--------|----------|-------------
lualzw|0.6622|0.0003|100
LibCompress|2.1983|0.0024|100

Input: 1000000 random generated bytes in ASCII range converted into string

algorithm|compress|decompress|result % of input
---------|--------|----------|-------------
lualzw|0.812|0.0022|100
LibCompress|1.782|0.0007|100

Input: 1000000 random generated repeating bytes converted into string

algorithm|compress|decompress|result % of input
---------|--------|----------|-------------
lualzw|0.3975|0.0262|4.5001
LibCompress|0.3907|0.0264|6.6997

Input: 1000000 of same character

algorithm|compress|decompress|result % of input
---------|--------|----------|-------------
lualzw|0.7045|0.0026|0.2829
LibCompress|0.6418|0.0038|0.4241

Input: "ymn32h8hm8ekrwjkrn9f" repeated 50000 times. In total 1000000 bytes

algorithm|compress|decompress|result % of input
---------|--------|----------|-------------
lualzw|0.4788|0.0088|1.2629
LibCompress|0.4426|0.0093|1.8905