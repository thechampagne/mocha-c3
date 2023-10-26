# mocha-c3

[![](https://img.shields.io/github/v/tag/thechampagne/mocha-c3?label=version)](https://github.com/thechampagne/mocha-c3/releases/latest) [![](https://img.shields.io/github/license/thechampagne/mocha-c3)](https://github.com/thechampagne/mocha-c3/blob/main/LICENSE)

A C3 binding for mocha parser an elegant configuration language for both humans and machines.

### Example
This program output all tokens in text.
```c
import mocha;
import libc;
import std::io;

fn int main() {
  char* text = `defaults: {
                  user_id: 0
                  start_id: user_id
                }
                hanna: {
                  name: 'hanna rose'
                  id: @:defaults:user_id
                  inventory: ['banana' 12.32]
                }`;

  Object obj;
  if (mocha::parse(&obj, text) != NONE) {
    return 1;
  }

  for (usz i = 0; i < obj.fields_len; i++) {
    Field field = obj.field(i);
    io::printf("%s: {\n", field.name);

    for (usz j = 0; j < field.value.object.fields_len; j++) {
      Field field0 = field.value.object.field(j);
      io::printf("%s: ", field0.name);
      switch (field0.type) {
        case INTEGER:
          io::printf("%ld\n", field0.value.integer64);
          break;
        case FLOAT:
          io::printf("%f\n", field0.value.float64);
          break;
        case STRING:
          io::printf("'%s'\n", field0.value.string);
          break;
        case ARRAY:
          io::print("[");
          for (usz idx = 0; idx < field0.value.array.items_len; idx++) {
            Value value;
            ValueType value_type = field0.value.array.array(&value, idx);
            if (value_type == STRING) {
              io::printf("'%s' ", value.string);
            } else if (value_type == FLOAT) {
              io::printf("%f", value.float64);
            }
          }
          io::print("]\n");
          break;
        case REFERENCE:
          libc::fwrite(field0.value.reference.name, field0.value.reference.name_len, 1, libc::stdout());
          if (field0.value.reference.child != null) io::print(":");
          Reference reference;
          reference.child = field0.value.reference.child;
          while (reference.next() == 0) {
            libc::fwrite(reference.name, reference.name_len, 1, libc::stdout());
            if (reference.child != null) io::print(":");
          }
          io::print("\n");
          break;
        default:
          break;
      }
    }
    io::print("}\n");
  }
  obj.deinit();
  return 0;
}
```

### References
 - [mocha](https://github.com/hqnna/mocha)
 - [libmocha](https://github.com/thechampagne/libmocha)

### License

This repo is released under the [BSD-3-Clause-Clear License](https://github.com/thechampagne/mocha-c3/blob/main/LICENSE).
