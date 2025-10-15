/**
 * LSB Steganography Demo (lsb_steganography.ts)
 *
 * Simple LSB embedding into a byte array (works with raw RGB buffers).
 * Demonstrates encode/decode of short messages without external image libs.
 *
 * Run: ts-node src/lsb_steganography.ts
 */

function embedMessage(buffer: Uint8Array, message: string) {
  const bits = [];
  for (let i = 0; i < message.length; i++) {
    const code = message.charCodeAt(i);
    for (let b = 7; b >= 0; b--) bits.push((code >> b) & 1);
  }
  // delimiter: 8 zeros
  bits.push(...Array(8).fill(0));
  if (bits.length > buffer.length) throw new Error('Buffer too small');
  const out = new Uint8Array(buffer);
  for (let i = 0; i < bits.length; i++) {
    out[i] = (out[i] & 0xFE) | bits[i];
  }
  return out;
}

function extractMessage(buffer: Uint8Array) {
  const bits: number[] = [];
  for (let i = 0; i < buffer.length; i++) bits.push(buffer[i] & 1);
  const bytes: number[] = [];
  for (let i = 0; i < bits.length; i += 8) {
    const slice = bits.slice(i, i + 8);
    if (slice.length < 8) break;
    const val = slice.reduce((s, b) => (s << 1) | b, 0);
    if (val === 0) break; // delimiter
    bytes.push(val);
  }
  return String.fromCharCode(...bytes);
}

// demo using synthetic RGB buffer
if (require.main === module) {
  const width = 100, height = 50;
  const pixels = width * height * 3;
  const buf = new Uint8Array(pixels).map(() => Math.floor(Math.random() * 256));
  const msg = 'hello stego';
  const encoded = embedMessage(buf, msg);
  const decoded = extractMessage(encoded);
  console.log('original:', msg, 'decoded:', decoded);
}
