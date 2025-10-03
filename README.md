# Lightweight Avatar Builder

A Vue 3 (Vite) application for building customizable avatars with compact 5-byte DNA encoding.

## Features

- Compact 5-byte avatar DNA encoding
- Base64 encoding for sharing
- Live SVG avatar preview
- Customizable avatar elements (hair, eyes, mouth, clothing, etc.)
- Randomize and load functionality

## Wire Format

The avatar DNA uses 5 bytes (40 bits) to encode all avatar properties:

```
Bit Range | Property       | Bits | Values
----------|----------------|------|--------
0-2       | Version        | 3    | 0-7 (currently v1)
3-5       | Skin Color     | 3    | 0-7
6-10      | Hair Style     | 5    | 0-31
11-14     | Eyes           | 4    | 0-15
15-18     | Mouth          | 4    | 0-15
19-22     | Facial Hair    | 4    | 0-15
23-25     | Hair Color     | 3    | 0-7
26-28     | Clothing Color | 3    | 0-7
29-32     | Clothing Style | 4    | 0-15
33-36     | Background     | 4    | 0-15
37-39     | Accessories    | 3    | 0-7
```

## Project Setup

```sh
npm install
npm run dev
npm run build
```


Check it out `https://rightup.github.io/LightweightAvatar/`

