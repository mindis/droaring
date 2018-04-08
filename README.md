# Experimental D Roaring Bitmaps Library

**EARLY RELEASE**

[Roaring Bitmaps](http://roaringbitmap.org) are compressed bit arrays which can store a huge amount of bits in a space efficient manner. The bitmap is organized so that adding/removing bits is very fast and don't require unpacking the whole bitmap. You can use bit arrays for efficient set operations.

Check out [Pilosa](https://www.pilosa.com) for an open source distributed index which uses roaring bitmaps.

This library wraps [CRoaring](https://github.com/RoaringBitmap/CRoaring).


## Requirements

* A recent D compiler. Tested with DMD v2.079.0 and LDC 1.8.0.
* Requirements for CRoaring (including a C compiler).
* Tested on Linux, FreeBSD and MacOS.

## Install

### Using DUB

Add `roaring` to your DUB dependencies. E.g.:
```json
{
    "name": "roar",
    "description": "A minimal D application.",
    "dependencies": {
        "roaring": {
            "version": "0.1.1"
        }
    }
}
```

### Without DUB

Assuming you've built the library and `$DROARING_HOME` points to the DRoaring directory:

```
dmd your_source.d $DROARING_HOME/ext/roaring.o -L-L$DROARING_HOME -L-lroaring -I$DROARING_HOME/source
```

## Example
```d
void main()
{
    import std.stdio : writefln, writeln;
    import roaring.roaring : Roaring, bitmapOf;

    // create a new roaring bitmap instance
    auto r1 = new Roaring;

    // add some bits to the bitmap
    r1.add(5);

    // create from an array
    auto ra = bitmapOf([1, 2, 3]);
    
    // create a new roaring bitmap instance from some numbers
    auto r2 = bitmapOf(1, 3, 5, 15);
    
    // check whether a value is contained
    assert(r2.contains(5));
    assert(5 in r2); // r2 contains 5
    assert(99 !in r2); // r2 does not contain 99

    // get minimum and maximum values in a bitmap
    assert(r2.minimum == 1);
    assert(r2.maximum == 15);

    // remove a value from the bitmap
    r2.remove(5);
    assert(!r2.contains(5));
    
    // compute how many bits there are:
    assert(3 == r2.length);

    // check whether a bitmap is subset of another
    const sub = bitmapOf(1, 5);
    const sup = bitmapOf(1, 2, 3, 4, 5, 6);
    assert(sub in sup);

    // iterate on a bitmap
    const r3 = bitmapOf(1, 5, 10, 20);
    ulong sum = 0;
    foreach (bit; r3) {
        sum += bit;
    }
    assert(sum == 36);

    // iterate on a bitmap and index
    foreach(i, bit; r3) {
        writefln("%d: %d", i, bit);
    }

    import roaring.roaring : readBitmap, writeBitmap;
    // serialize the bitmap
    char[] buf = writeBitmap(r3);
    // deserialize from a char array
    const r3Copy = readBitmap(buf);
    assert(r3 == r3Copy);

    // find the intersection of bitmaps
    const r4 = bitmapOf(1, 5, 6);
    const r5 = bitmapOf(1, 2, 3, 4, 5);
    assert((r4 & r5) == bitmapOf(1, 5));
    // find the union of bitmaps
    assert((r4 | r5) == bitmapOf(1, 2, 3, 4, 5, 6));

    const r6 = bitmapOf(0, 10, 20, 30, 40, 50);
    // get the bit for the index
    assert(20 == r6[2]);
    // slice a bitmap
    assert(bitmapOf(30, 40, 50) == r6[3..$]);

    // convert the bitmap to a string
    writeln("Bitmap: ", r6);
    import std.conv : to;
    assert("{0, 10, 20, 30, 40, 50}" == to!string(r6));
}
```

## Build

### Using DUB

Using default D compiler:

```
dub build
```

Specifying the D compiler:
```
dub build --compiler=$LDC_HOME/bin/ldc2
```

### Using make

Using default D compiler:

```
make
```

Specifying the D compiler:
```
make DMD=$LDC_HOME/bin/ldc2
```

## Running Tests

```
dub test
```

## License

* `ext/roaring.c` and `ext/roaring.h` were generated from [CRoaring](https://github.com/RoaringBitmap/CRoaring/) source. Copyright 2016 The CRoaring authors. See the [LICENSE](https://github.com/RoaringBitmap/CRoaring/blob/master/LICENSE).

* D Wrapper for CRoaring: Copyright 2018 Yüce Tekol. See the [LICENSE](https://github.com/yuce/droaring/blob/master/LICENSE).
