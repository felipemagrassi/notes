# Tracing ruby code with RubyVM - 2023-05-15 00:30:07.078732859 -0300 -03 m=+2783.910504823

We can #trace #ruby code with `RubyVM::InstructionSequence` class.

```ruby
code = <<EOL
index = 42
page_size = 10
index / page_size
index % page_size
EOL

puts RubyVM::InstructionSequence.compile(code).disasm

code = <<EOL 
index = 42
page_size = 10
index.divmod(page_size)
EOL

puts RubyVM::InstructionSequence.compile(code).disasm
```

*Results:*

    == disasm: #<ISeq:<compiled>@<compiled>:1 (1,0)-(4,17)> (catch: FALSE)
    local table (size: 2, argc: 0 [opts: 0, rest: -1, post: 0, block: -1, kw: -1@-1, kwrest: -1])
    [ 2] index@0    [ 1] page_size@1
    0000 putobject                              42                        (   1)[Li]
    0002 setlocal_WC_0                          index@0
    0004 putobject                              10                        (   2)[Li]
    0006 setlocal_WC_0                          page_size@1
    0008 getlocal_WC_0                          index@0                   (   3)[Li]
    0010 getlocal_WC_0                          page_size@1
    0012 opt_div                                <calldata!mid:/, argc:1, ARGS_SIMPLE>
    0014 pop
    0015 getlocal_WC_0                          index@0                   (   4)[Li]
    0017 getlocal_WC_0                          page_size@1
    0019 opt_mod                                <calldata!mid:%, argc:1, ARGS_SIMPLE>
    0021 leave
    == disasm: #<ISeq:<compiled>@<compiled>:1 (1,0)-(3,23)> (catch: FALSE)
    local table (size: 2, argc: 0 [opts: 0, rest: -1, post: 0, block: -1, kw: -1@-1, kwrest: -1])
    [ 2] index@0    [ 1] page_size@1
    0000 putobject                              42                        (   1)[Li]
    0002 setlocal_WC_0                          index@0
    0004 putobject                              10                        (   2)[Li]
    0006 setlocal_WC_0                          page_size@1
    0008 getlocal_WC_0                          index@0                   (   3)[Li]
    0010 getlocal_WC_0                          page_size@1
    0012 opt_send_without_block                 <calldata!mid:divmod, argc:1, ARGS_SIMPLE>
    0014 leave
