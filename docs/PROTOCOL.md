# PROTOCOL (Client <-> Server)

Version: v1.0
Transport: TCP
Encoding: JSON per line (one JSON object per request, ended by '\n')
Binary: base64 in JSON field for file data chunks

## Common response
{"ok":true,"code":0,"msg":"OK","data":{}}

Error example:
{"ok":false,"code":-2,"msg":"ENOENT","data":{}}

## Auth

### PING
Req:
{"op":"PING"}
Resp:
{"ok":true,"code":0,"msg":"PONG","data":{"version":"v1.0"}}

### LOGIN
Req:
{"op":"LOGIN","user":"alice","password":"123456"}
Resp:
{"ok":true,"code":0,"msg":"OK","data":{"token":"t_xxx","role":"author"}}

### LOGOUT
Req:
{"op":"LOGOUT","token":"t_xxx"}
Resp:
{"ok":true,"code":0,"msg":"OK","data":{}}

## VFS-backed ops (map to VfsAPI)

### MKDIR
{"op":"MKDIR","token":"t_xxx","path":"/papers"}

### LIST
Req:
{"op":"LIST","token":"t_xxx","path":"/papers"}
Resp:
{"ok":true,"code":0,"msg":"OK","data":{"names":["paper_a1b2c3"]}}

### READ (chunked download)
Req:
{"op":"READ","token":"t_xxx","path":"/papers/paper_a1b2c3/v1.pdf","off":0,"n":4096}
Resp:
{"ok":true,"code":0,"msg":"OK","data":{"off":0,"n":1234,"eof":false,"b64":"AAAA..."}}

### WRITE (chunked upload)
Req:
{"op":"WRITE","token":"t_xxx","path":"/papers/paper_a1b2c3/v1.pdf","off":0,"b64":"AAAA..."}
Resp:
{"ok":true,"code":0,"msg":"OK","data":{"written":4096}}
