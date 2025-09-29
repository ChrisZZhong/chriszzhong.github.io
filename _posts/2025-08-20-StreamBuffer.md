---
title: "StreamBuffer"
date: 2025-06-30
description: "StreamBuffer"
tag: algorithms
prime: false
---

## Scanner

try to aviod using scanner, because read each line contains an I/O operation.

BufferReader will preload a batch of data. (efficient)

```Java
// 此种情况适用于按token读取，如果有换行符和空格则无法区别行
public static int[][] mat;

public static void main() {
    BufferReader br = new BufferReader(new InputStramReader(System.in));
    StreamTokenizer in = new StreamTokenizer(br);

    PrintWriter out = new PrintWriter(new OutputStreamWriter(System.out));

    while (in.nextToken() != StreamTokenizer.TT_EOF) {
        int n = (int) in.val;
        in.nextToken();
        int m = (int) in.val;
        mat = new int[n][m];
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                in.nextToken();
                mat[i][j] = (int) in.val;
            }
        }
        out.println(method());
    }
    out.flush();
    out.close();
}

```

```java
// 
public static String line;
public static void main() {
    BufferReader br = new BufferReader(new InputStramReader(System.in));
    PrintWriter out = new PrintWriter(new OutputStreamWriter(System.out));

    while ((line = in.readLine()) != null) {
        parts = line.split(" ");
        out.println(method());
    }
    out.flush();
    in.close();
    out.close();
}

```
