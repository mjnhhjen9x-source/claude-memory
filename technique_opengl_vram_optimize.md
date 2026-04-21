---
name: technique_opengl_vram_optimize
description: Toi uu VRAM cho game OpenGL (PC) - noop texture upload + glDeleteTextures tu render thread
type: reference
originSessionId: d3db5cfb-3319-4f96-8d5d-4df12d7f4943
---
# Tối ưu VRAM Game OpenGL (PC)

## Áp dụng cho
Game PC dùng OpenGL (MU Online, các game engine cũ PE32 x86 Windows).
Đã xác nhận hoạt động trên M4VN (optimize.js v3).

## Vấn đề
- FPS cap (wglSwapBuffers Sleep) + render disable (glClear/glDrawArrays noop) → CPU/GPU giảm
- Nhưng VRAM vẫn cao vì `glTexImage2D`/`glTexSubImage2D` **không bị chặn** → game vẫn upload texture lên VRAM
- Texture cũ đã upload trước khi render tắt vẫn nằm trong VRAM

## Giải pháp (3 phần)

### 1. Track texture ID qua glGenTextures hook
```javascript
var trackedIds = {};
Interceptor.attach(opengl.getExportByName("glGenTextures"), {
    onEnter: function(args) { this.n = args[0].toInt32(); this.idsp = args[1]; },
    onLeave: function() {
        for (var i = 0; i < this.n; i++) {
            var id = this.idsp.add(i * 4).readU32();
            if (id > 0) trackedIds[id] = 1;
        }
    }
});
```

### 2. Noop glTexImage2D + glTexSubImage2D khi render tắt
```javascript
var uploadNoop = false;
var orig2D = new NativeFunction(opengl.getExportByName("glTexImage2D"), "void",
    ["uint32","int32","int32","int32","int32","int32","uint32","uint32","pointer"], "stdcall");
Interceptor.replace(opengl.getExportByName("glTexImage2D"), new NativeCallback(
    function(target,level,ifmt,w,h,border,fmt,type,data) {
        if (!uploadNoop) orig2D(target,level,ifmt,w,h,border,fmt,type,data);
    }, "void", ["uint32","int32","int32","int32","int32","int32","uint32","uint32","pointer"], "stdcall"
));
// Tương tự cho glTexSubImage2D (9 args: target,level,xoff,yoff,w,h,fmt,type,data)
```

### 3. glDeleteTextures gọi từ render thread
**QUAN TRỌNG**: glDeleteTextures BẮT BUỘC gọi từ thread đang có OpenGL context active.
Dùng flag `pendingFree` → check trong wglSwapBuffers hook.

```javascript
var pendingFree = false;
var DeleteTextures = new NativeFunction(
    opengl.getExportByName("glDeleteTextures"), "void", ["int32","pointer"], "stdcall");

function doFreeVram() {
    var ids = Object.keys(trackedIds);
    if (ids.length === 0) return 0;
    var buf = Memory.alloc(ids.length * 4);
    for (var i = 0; i < ids.length; i++) buf.add(i*4).writeU32(parseInt(ids[i]));
    DeleteTextures(ids.length, buf);
    trackedIds = {};
    return ids.length;
}

// Trong wglSwapBuffers onLeave:
Interceptor.attach(opengl.getExportByName("wglSwapBuffers"), {
    onLeave: function(retval) {
        if (pendingFree) { pendingFree = false; doFreeVram(); }
        Sleep(frameDelay);
    }
});
```

### 4. Kết hợp khi disableRender()
```javascript
function disableRender() {
    // ... noop glClear, glDrawArrays, glDrawElements
    uploadNoop = true;                              // chặn texture upload mới
    setTimeout(function() { pendingFree = true; }, 3000);   // xóa texture cũ sau 3s
    setInterval(function() { pendingFree = true; }, 60000); // dọn lại mỗi 60s
}
```

## Lưu ý
- `glDeleteTextures` từ thread khác → crash hoặc không có tác dụng
- Sau khi delete, texture ID vẫn tồn tại trong game nhưng không có data → không vẽ được (OK vì render đã tắt)
- `glTexSubImage2D` có 9 args giống `glTexImage2D` nhưng thêm xoffset/yoffset (update vùng texture)
- File hoàn chỉnh: `D:\Code Tools\PC_GAMES\M4VN\optimize.js` (v3)
