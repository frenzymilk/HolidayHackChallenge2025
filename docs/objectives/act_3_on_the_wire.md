---
icon: material/text-box-outline
---

# Act 3: On the Wire

**Difficulty**: :fontawesome-solid-star::fontawesome-solid-star::fontawesome-solid-star::fontawesome-solid-star::fontawesome-regular-star:<br/>


## Objective

!!! question "Request"
    Help Evan next to city hall hack this gnome and retrieve the temperature value reported by the I²C device at address 0x3C. The temperature data is XOR-encrypted, so you’ll need to work through each communication stage to uncover the necessary keys. Start with the unencrypted data being transmitted over the 1-wire protocol.

??? quote "Evan Booth"
    So here's the deal - there are some seriously bizarre signals floating around this area.<br/>
    Not your typical radio chatter or WiFi noise, but something... different.<br/>
    I've been trying to make sense of the patterns, but it's like trying to build a robot hand out of a coffee maker - you need the right approach.<br/>
    Think you can help me decode whatever weirdness is being transmitted out there?<br/>
    You know what happens to electronics in extreme cold? They fail. All my builds, all my robots, all my weird coffee-maker contraptions—frozen solid. We can't let Frosty turn this place into a permanent deep freeze.<br/>

## Hints

??? tip "Structure"
    What you're dealing with:<br/>
    You have access to WebSocket endpoints that stream digital signal data
    Each endpoint represents a physical wire in a hardware communication system
    The data comes as JSON frames with three properties: line (wire name), t (timestamp), and v (value: 0 or 1)
    The server continuously broadcasts signal data in a loop - you can connect at any time
    This is a multi-stage challenge where solving one stage reveals information needed for the next <br/>
    Where to start:<br/>
    Connect to a WebSocket endpoint and observe the data format
    The server automatically sends data every few seconds - just wait and collect
    Look for documentation on the protocol types mentioned (1-Wire, SPI, I2C)
    Consider that hardware protocols encode information in the timing and sequence of signal transitions, not just the values themselves
    Consider capturing the WebSocket frames to a file so you can work offline

## Solution
We are faced with a digital signal that is encoded following different protocols successively: 1-Wire, SPI and I2C.
We have to decode the signal for each of these protocols before we are able to reach the temperature instruction.

I wrote different scripts in order to do that.
First, let's decode the frames received on the 1-Wire protocol

```python
import json

binary = ""
start_read = False
t0=None
t1=None

def binary_to_string(binary_string):
    bytes_list = [binary_string[i:i+8][::-1] for i in range(0, len(binary_string), 8)]
    # Convert each 8-bit chunk to its integer ASCII value, then to a character
    characters = [chr(int(byte_chunk, 2)) for byte_chunk in bytes_list]
    
    print(''.join(characters))

with open("1wire.txt") as f:

    for json_txt in f:

        json_value = json.loads(json_txt.strip())

        if json_value.get("marker"):
            if json_value.get("marker") == "stop":
                break
            if json_value.get("marker") == "presence":
                start_read = True
                continue
        if start_read and json_value.get("t")!=701:
            if t0 == None and t1 == None:
                t0 = json_value.get("t")
                continue
            if t0 and t1 == None:
                t1 = json_value.get("t")

                pulse = t1 - t0

                if pulse < 10: #1
                    binary += "1"
                else:          #0
                    binary += "0"
                t0 = None
                t1 = None



print(binary)
print(len(binary))
print(binary_to_string(binary))

'''
SOLUTION:
001100110100111010100110100001100010011000000100100001100111011000100110000001000010011010100110110001100100111010011110000011100010111000000100001011100001011010100110000001001100101000001010100100100000010001000110101011101100111000000100001001101000011000101110100001100000010010101110110011101001011001110110111001100000010000101110000101101010011000000100000110101111001001001010000001001101011010100110100111100101110000000100100101101100011010011110
456
Ìread and decrypt the SPI bus data using the XOR key: icy
'''

```

Next, the SPI data:

```python
import json

start_read = False
start_mosi_read = False
sck_list = list()
mosi_list = list()
binary = ""


def string_to_binary(input_string):
    # '08b' format specifier ensures each character's binary representation 
    # is at least 8 bits long and padded with leading zeros.
    return ''.join(format(ord(c), '08b') for c in input_string)

def xor_bits(data_bits, key_bits):
    out = []
    for i, bit in enumerate(data_bits):
        kb = key_bits[i % len(key_bits)]
        out.append('1' if bit != kb else '0')
    return ''.join(out)



with open("sck.txt") as f:
    for json_txt in f:
        json_value = json.loads(json_txt.strip())

        if json_value.get("marker") == "idle-low" and json_value.get("t") == 0:
            start_read = True
            continue

        if start_read and json_value.get("marker") == "idle-low" and json_value.get("t") == 8000000:
            break

        if start_read:
            if json_value.get("marker") == "sample":
                sck_list.append(json_value)

with open("mosi.txt") as f:
    for json_txt in f:
        json_value = json.loads(json_txt.strip())

        if json_value.get("marker") == "idle-low" and json_value.get("t") == 0:
            start_mosi_read = True
            continue

        if start_mosi_read and json_value.get("marker") == "data-bit" and json_value.get("t") == 7990000:
            mosi_list.append(json_value)
            break

        if start_mosi_read:
            mosi_list.append(json_value)

for elmt in sck_list:
    current_clock =  elmt.get("t")
    for elmt_mosi in mosi_list:
        t_mosi = elmt_mosi.get("t")
        if t_mosi <= current_clock:
            current_mosi = elmt_mosi.get("v")
        else:
            binary += str(current_mosi)
            break


binary_key = string_to_binary("icy")
xored_binary = xor_bits(binary, binary_key)

print(xored_binary)

bytes_list = [xored_binary[i:i+8] for i in range(0, len(xored_binary), 8)]
# Convert each 8-bit chunk to its integer ASCII value, then to a character
characters = [chr(int(byte_chunk, 2)) for byte_chunk in bytes_list]

print(''.join(characters))
'''
SOLUTION:
0111001001100101011000010110010000100000011000010110111001100100001000000110010001100101011000110111001001111001011100000111010000100000011101000110100001100101001000000100100100110010010000110010000001100010011101010111001100100000011001000110000101110100011000010010000001110101011100110110100101101110011001110010000001110100011010000110010100100000010110000100111101010010001000000110101101100101011110010011101000100000011000100110000101101110011000010110111001111010011000010010111000100000011101000110100001100101001000000111010001100101011011010111000001100101011100100110000101110100011101010111001001100101001000000111001101100101011011100111001101101111011100100010000001100001011001000110010001110010011001010111001101110011001000000110100101110011001000000011000001111000001100110100001
read and decrypt the I2C bus data using the XOR key: bananza. the temperature sensor address is 0x3!
'''
```

And finally, the I2C data

```python
import json
import math

start_slc_read = False
start_sda_read = False
scl_list = list()
sda_list = list()
binary = ""


def string_to_binary(input_string):
    # '08b' format specifier ensures each character's binary representation 
    # is at least 8 bits long and padded with leading zeros.
    return ''.join(format(ord(c), '08b') for c in input_string)

def xor_bits(data_bits, key_bits):
    out = []
    for i, bit in enumerate(data_bits):
        kb = key_bits[i % len(key_bits)]
        out.append('1' if bit != kb else '0')
    return ''.join(out)



with open("sck.txt") as f:
    for json_txt in f:
        json_value = json.loads(json_txt.strip())
        scl_list.append(json_value)

with open("sda.txt") as f:
    for json_txt in f:
        json_value = json.loads(json_txt.strip())
        sda_list.append(json_value)

addresses = list()
read_write_bit = None
data = dict()
count_address = 0
count_frame = 0
count_instruction = 0

for element in sda_list:
    if element.get("marker") == "address-bit" and element.get("bitIndex") != 7:
        
        data.setdefault("address-"+ str(math.floor(count_address/7)), "" )
        data["address-"+ str(math.floor(count_address/7))]+= str(element.get('v'))
        count_address += 1

    if element.get("marker") == "address-bit" and element.get("bitIndex") == 7:
        data["instruction-" + str(count_instruction)] = str(element.get("v"))
        count_instruction += 1

    if element.get("marker") == "gap-start":
        count_frame += 1

    if element.get("marker") == "data-bit":
        data.setdefault("data-"+str(count_frame), "")
        data["data-"+str(count_frame)] += str(element.get("v"))


binary_key = string_to_binary("bananza")

for element in data:
    if "addr" in element:
        data[element] = hex(int(data[element],2))

    if "data" in element:
        xored_binary = xor_bits(data[element], binary_key)
        bytes_list = [xored_binary[i:i+8] for i in range(0, len(xored_binary), 8)]
        # Convert each 8-bit chunk to its integer ASCII value, then to a character
        characters = [chr(int(byte_chunk, 2)) for byte_chunk in bytes_list]

        txt = ''.join(characters)
        data[element] = str(txt)



print(data)
'''
{'address-0': '0x48', 'instruction-0': '0', 'data-0': '45%', 'address-1': '0x3c', 'instruction-1': '0', 'data-1': '32.84', 'address-2': '0x51', 'instruction-2': '0', 'data-2': '1013 hPa', 'address-3': '0x29', 'instruction-3': '0', 'data-3': '450 lux'}
'''

```

##### Final decryption
The temperature we are looking for is **32.84**