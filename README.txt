# OpenGL 3D Renderer

A real-time 3D rendering engine written in C++ with OpenGL, GLAD, and GLFW, built while following Victor Gordan's [OpenGL Tutorial for Beginners](https://www.youtube.com/playlist?list=PLPaoO-vpZnumdcb4tZc4x5Q-v7CkrQ6M-) series. The goal was a solid understanding of the real-time rendering pipeline, shader programming, and GPU resource management, built up through lighting, texturing, and glTF model loading.

## Notes: how the buffers fit together

### VBO (Vertex Buffer Object)

Raw per-vertex data stored in GPU memory, interleaved by attribute:

```
Index | Vertex data
------+------------------------------------
  0   | Position(0)  Color(0)  TexUV(0)   <- referenced by EBO
  1   | Position(1)  Color(1)  TexUV(1)   <- referenced by EBO
  2   | Position(2)  Color(2)  TexUV(2)   <- referenced by EBO
  3   | Position(3)  Color(3)  TexUV(3)   <- referenced by EBO
  4   | Position(4)  Color(4)  TexUV(4)   <- unused (for now)
  5   | Position(5)  Color(5)  TexUV(5)   <- unused (for now)
```

### EBO (Element Buffer Object)

Indices into the VBO, so shared vertices are stored once and reused:

```
[ 0, 1, 2,   2, 3, 0 ]  ->  two triangles forming a quad
```

### VAO (Vertex Array Object)

Records the EBO binding and how each attribute is read from the VBO:

```
VAO #1
├── EBO = #7
├── Attribute 0 (position)
│   ├── VBO     = #5
│   ├── size    = 3
│   ├── type    = GL_FLOAT
│   ├── stride  = 44 bytes
│   ├── offset  = 0
│   └── enabled = true
└── (other attributes disabled)
```

Without a VAO, every draw call would need the full setup again:

```cpp
glBindBuffer(GL_ARRAY_BUFFER, vbo);
glVertexAttribPointer(...);   // position
glVertexAttribPointer(...);   // color
glVertexAttribPointer(...);   // texUV
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, ebo);
```

With a VAO, all of that state is captured once and restored with:

```cpp
vao.Bind();
```
