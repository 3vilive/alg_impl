# varint and zigzag

## varint

```python
VARINT_DATA_BYTE_BITS = 0x7F
VARINT_EXISTS_NEXT_BYTE_BIT = 0x80


def varint_encode(encode_val):
    byte_arr = bytearray()

    while True:
        varint_byte = encode_val & VARINT_DATA_BYTE_BITS
        rest_val = encode_val >> 7

        byte_arr.append(varint_byte)

        if not rest_val:
            break

        encode_val = rest_val

    byte_arr.reverse()
    for offset, byte in enumerate(byte_arr):
        if offset + 1 == len(byte_arr):
            # last byte
            break

        byte_arr[offset] = byte_arr[offset] | VARINT_EXISTS_NEXT_BYTE_BIT

    return byte_arr


def varint_decode(varint_byte_arr):
    # type: (bytearray) -> int
    decode_val = 0x0

    for offset, byte in enumerate(varint_byte_arr):
        byte_data = byte & VARINT_DATA_BYTE_BITS
        decode_val = decode_val + byte_data

        if offset + 1 == len(varint_byte_arr):
            # last byte
            break

        else:
            decode_val = decode_val << 7

    return decode_val

def test_varint():
    varint_test_list = [1235, 99976, 12, 45]
    for test_val in varint_test_list:
        encoded = varint_encode(test_val)
        print('encoded of {}: {}'.format(test_val, ' '.join(map(bin, encoded))))
        print('decode: {}'.format(varint_decode(encoded)))

if __name__ == '__main__':
    test_varint()
```

result:

```
encoded of 1235: 0b10001001 0b1010011
decode: 1235
encoded of 99976: 0b10000110 0b10001101 0b1000
decode: 99976
encoded of 12: 0b1100
decode: 12
encoded of 45: 0b101101
decode: 45
```

## zigzag

```python

def varint_decode(varint_byte_arr):
    # type: (bytearray) -> int
    decode_val = 0x0

    for offset, byte in enumerate(varint_byte_arr):
        byte_data = byte & VARINT_DATA_BYTE_BITS
        decode_val = decode_val + byte_data

        if offset + 1 == len(varint_byte_arr):
            # last byte
            break

        else:
            decode_val = decode_val << 7

    return decode_val


def zigzag_encode(int_val):
    """
     0 => 0
    -1 => 1
     1 => 2
    -2 => 3
     2 => 4
    -3 => 5
     3 => 6
    """
    if int_val >= 0:
        return int_val << 1
    else:
        return (abs(int_val) << 1) - 1

def zigzag_decode(zigzag_code):
    if zigzag_code % 2 == 0:
        return zigzag_code >> 1
    else:
        return -(zigzag_code >> 1) - 1


def test_zigzag():
    zigzag_test = [0, -1, 1, -2, 2, -3, 3, 555, 444, -123213]
    for original in zigzag_test:
        encode_val = zigzag_encode(original)
        print('{} => {}'.format(original, encode_val))
        print('{} => {} (decode)'.format(encode_val, zigzag_decode(encode_val)))

if __name__ == '__main__':
    test_zigzag()
```

output:

```
0 => 0
0 => 0 (decode)
-1 => 1
1 => -1 (decode)
1 => 2
2 => 1 (decode)
-2 => 3
3 => -2 (decode)
2 => 4
4 => 2 (decode)
-3 => 5
5 => -3 (decode)
3 => 6
6 => 3 (decode)
555 => 1110
1110 => 555 (decode)
444 => 888
888 => 444 (decode)
-123213 => 246425
246425 => -123213 (decode)
```