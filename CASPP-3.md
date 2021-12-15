1.  x86-64 寄存器     
    + 程序计数器，通常成为 PC，给出 ___给出将要执行的下一条指令___ 的内存地址      
    + 整数寄存器文件，包含16个命名的位置，分别存储64位的值，可以存储：      
      + 存储地址      
      + 整数数据      
      + 记录某些重要的程序状态    
      + 保存临时数据，例如过程的参数，局部变量，或者返回值    
      + 条件码寄存器，保存最近执行的算数或者逻辑指令的状态信息，它们用来实现控制或者数据流中的条件，比如说实现 `if`，`while` 语句   
      + 一组向量寄存器可以存放一个或者多个整数或者浮点数值      
2.  x86-64 机器指令     
    1.  x86-64 的指令长度从1-15个字节不等     
    2.  指令格式：从某个给定位置开始，将字节唯一地解码成机器指令      
    3.  反汇编基于机器代码文件中的字节序列确定汇编代码，它不需要访问该程序的源代码或者汇编代码      
    4.  反汇编器使用的指令明明规则与 GCC 产生的汇编代码使用有些细微的差别。它省略了很多指令结尾的 'q' ，这些后缀时大小指示符。在大多数情况下都可以省略     
3.  所有以 '.' 开头的行都是指导汇编器和连接器工作的伪指令   
4.  后缀表示的字节数：      
    |后缀|字节数|
    |-|-|
    |b|1|
    |w|2|
    |l|4|
    |q|8|

5.  ![alt x86-64 的整数寄存器](./pictures/整数寄存器.png "x86-64 的整数寄存器")     
    + 16个寄存器的低位部分都可以作为字节，字，双字，四字来访问      
    + 访问小于8字节结果的指令时，有两种规则：    
      + 生成1字节和2字节数字的指令会保持剩下的字节不变      
      + 生成4字节数字的指令会把高位4字节置为0，这条规则时从 IA32 到x86-64 的扩展的一部分采用的      
6.  操作数，大多数指令有一个或者多个操作数，它用来指示出执行一个操作中要使用的源数据值，以及放置结果的目的位置    
    + 源操作数可以以常数给出，或者从寄存器，内存中读出。结果可以存放在寄存器或者内存中，因此操作数可能性可以被分为三种类型：      
      + 立即数，表示常数值。不同指令允许的立即数数值范围不同，汇编器会自动选择最近凑的方式进行数值编码      
      + 寄存器，它表示某个寄存器的内容，16个寄存器的低位。用 r<sub>a</sub> 表示寄存器a，用 R[r<sub>a</sub>] 表示它的值    
      + 内存引用，会根据计算出来的地址访问某个内存的位置。用符号 M<sub>a</sub>[Addr] 表示对存储在内存中从地址 Addr 开始的 b 个字节的引用。   
7.  练习题3-1： 9(%rax, %rax) = M[R[rax] + R[rax] + 9]      
8.  简单的数据传送指令    

    |指令|效果|描述|
    |-|-|-|
    |MOV S, D|D &larr; S|传送| 
    |movb||传送字节|
    |movw||传送字|
    |movl||传送双字|
    |movq||传送四字|
    |movabsq I, R|R &larr; I|传送绝对的四字|
    + 源操作数可以存储在内存或者寄存器中；目的操作数指定一个位置，要么在内存中，或者在寄存器中。但两者不能同时都 ___指向内存位置___         
    + mov 指令只会更新目的操作数指定的那些寄存器字节或者内存位置。唯一例外的是，mvol 指令以寄存器作为目的时。他会将该寄存器的高位4字节设置为0。原因是 x86-64 采用惯例，即任何为寄存器生成32位值的指令都会把该寄存器的高位部分置为0        
    + 五种组合      
        ```
      movl $0x4050, %eax         立即数 -> 寄存器，4 bytes
      movw %bp, %sp              立即数 -> 寄存器，2 bytes
      movb (%rdi, %rcx), %al     内存 -> 寄存器，1 bytes
      movb $-17, (%rsp)          立即数 -> 内存，1 bytes
      movq %rax, -12(%rbp)       寄存器 -> 内存，8 bytes
        ```
      上述的 `movq` 指令只能以表示为32为补码数字的立即数作为源操作数，然后把这个值符号扩展到64位的值，放到目的位置。`movabsq` 指令能够以任意64位立即数值作为源操作数，并且只能以寄存器作为目的     
9.  `movz` 类中的指令把目的中剩余的字节填充为0。每条指令名字的最后两个字符都是大小指示符，第一个字符指定源的大小，第二个指定目的的大小     
    |指令|效果|描述|
    |-|-|-|
    |movz S, R| R &larr; 零扩展(S)|以零扩展进行传送|
    |movzbw||将做了零扩展的字节传送到字|
    |movzbl||将做了零扩展的字节传送到双字|
    |movzwl||将做了了零扩展的字传送到双字|
    |movzbq||将做了零扩展的字节传送到四字|
    |movzwq||将做了零扩展的字传送到四字|

10. `movs` 类中的指令通过符号扩展来填充，把源操作的最高位进行复制。每条指令名字的最后两个字符都是大小指示符，第一个字符指定源的大小，第二个指明目的的大小       
    |指令|效果|描述|
    |-|-|-|
    |movs S, R|R &larr; 符号扩展(S)|传送符号扩展的字节|
    |movsbw||将做了符号扩展的字节传送到字|
    |movsbl||将做了符号扩展的字节传送到双字|
    |movswl||将做了符号扩展的字传送到双字|
    |movsbq||将做了符号扩展的字节传送到四字|
    |movswq||将做了符号扩展的字传送到四字|
    |movslq||将做了符号扩展的双字传送到四字|
    |cltq|%rax &larr; 符号扩展(%eax)|把%eax符号扩展到%rax|

11. 练习题3-3     
    ```
    movb $0xF, (%ebx)               can not use %ebx as address register
    movl %rax, (%rsp)               mismatch between instruction suffix and register ID
    movw (%rax), 4(%rsp)            can not have both source and destination be memory references
    movb %al, %sl                   no register named %sl
    movq %rax, $0x123               can not have immedinate as distination
    movl %eax, %rdx                 destination operand incorrect size
    movb %si, 8(%rbp)               mismatch between instruction suffix and register ID
    ```
12. 练习题3-4，假设：     
    ```
    src_t *sp;
    dest_t *dp;
    ```
    要实现 `*dp = (dest_t)*sp;`以下变换：     
    ___当执行强制类型转换既设计大小变换又涉及 C语言 中符号变化时，操作应该先改变大小___     

    |src_t|dest_t|指令|
    |-|-|-|
    |long|long|movq (%rdi), %rax<br>movq %rax, (%rsi)|
    |char|int|movsbl (%rdi), %eax<br>movl %eax, (%rsi)|
    |char|unsigned|movsbl (%rdi), %eax<br>movl %eax, (%rsi)|
    |unsigned char|long|movzbl %(rdi), %eax<br>movq %rax, (%rsi)|
    |int|char|movl (%rdi), %eax<br>movb %al, (%rsi)|
    |unsigned|unsigned char|movl (%rdi), %eax<br>movb (%al), (%rsi)|
    |char|short|movsbw (%rdi), %ax<br>movw %ax, (%rsi)|

13. `push` 和 `pop` 参考另一个《汇编原理》       
14. 算术和逻辑操作分为四种：   
    + 加载有效地址      
    + 一元操作    
    + 二元操作，目的操作数如果是内存地址时，处理器必须从内存读出地址，执行操作，再把结果写回内存    
    + 移位      

    |指令|效果|描述|
    |-|-|-|
    |leaq S, D|D &larr; &S|加载有效地址|
    |INC D|D &larr; D + 1|加1|
    |DEC D|D &larr; D - 1|减1|
    |NEG D|D &larr; -D|取负|
    |NOT D|D &larr; ~D|取补|
    |ADD S, D|D &larr; D + S|加|
    |SUB S, D|D &larr; D - S|减|
    |IMUL S, D|D &larr; D * S|乘|     
    |XOR S, D|D &larr; D ^ S|异或|      
    |OR S, D|D &larr; D \| S|或|
    |AND S, D|D &larr; D & S|与|
    |SAL k, D|D &larr; D << k|算术左移|
    |SHL k, D|D &larr; D << k|逻辑左移，等同于 SAL|
    |SAR k, D|D &larr; D >> k|算术右移|
    |SHR k, D|D &larr; D >> k|逻辑有移|

15. leaq(load effective address)：      
    + 从内存读数据到寄存器，但实际上它根本就没有引用内存。它的第一个操作数看上去是一个内存引用，但是该指令并不是从指定的位置读入数据，而是将有效地址写入到目的操作数     
    + 如果寄存器 `%rax` 的值位x，那么指令 `leaq 7(%rdx, %rdx, 4), %rax` 将设置寄存器 `%rax` 的值位 `5x + 7`。这种情况下目的操作数必须是一个寄存器     
16. 移位量可以是一个立即数，或者放在单字节寄存器 `%cl` 中。   
    + 左移/右移的效果是一样的，在右边补0.算术移位补上符号位，逻辑移位补上0.   
    + 移位操作的目的操作数可以是寄存器或者是内存位置      
17. 两个 64 位有符号或者无符号整数相等得到的乘积需要128位表示     
    |指令|效果|描述|
    |-|-|-|
    |imulq S<br>mulq S|R[%rax]: R[%rax] &larr; S X R[%rax]<br>R[%rax]: R[%rax] &larr; S X R[%rax]|有符号乘法<br>无符号乘法|
    |clto|R[%rax]: R[%rax] &larr; 符号扩展(R[%rax])|转换八字|
    |idivq S|R[%rdx] &larr; R[%rdx]: R[%rax] mod S<br>R[%rdx] &larr; R[%rdx]: R[%rax] / S|有符号除法|
    |divq S|R[%rdx] &larr; R[%rdx]: R[%rax] mod S<br>R[%rdx] &larr; R[%rdx]: R[%rax] / S|无符号除法|

    + 这种但操作数的乘法，计算的是两个64位值的全128位乘积。要求：     
      + 一个操作数在寄存器%rax中    
      + 一个操作数作为源操作数给出      
      + 乘积高位存放在%rdx中      
      + 乘积低位存放在%rax中      
    + 有符号除法操作类似，要求：    
      + 将寄存器的高位放到%rdx      
      + 将寄存器的低位放到%rax(%rdx + %rax 存储的是被除数)      
      + 除数作为指令的操作数给出      
      + 商存储到%rax中      
      + 余数存储到寄存器%rdx中      
      + 对于大多数64位除法应用来说，除数也常常是64位的值。这个值存在%rax，%rdx的位应该设置为全0或者%rax的符号位。后面的操作使用cqto来完成，它隐含读出%rax的符号位，并将它复制到%rdx的所有位       
      + 无符号除法使用divq指令，通常%rdx会事先设置为0       
18. 条件码寄存器:     
    + 用于描述了最近的 __算术__ 或者 __逻辑__ 操作。        
    + 可以检测这些寄存器来执行条件分支          
    + 最常用的条件码：      
      + CF，进位标志，最近操作使最高位产生了进位。可用来检测无符号的操作溢出      
      + ZF，零标志位，最近的操作得出的结果是0     
      + SF，符号标志，最近的操作得到的结果是负数      
      + OF，溢出标志，最近的操作导致一个补码溢出 -- 正溢出或者负溢出        
      + 常见的操作影响条件码：      
        + leaq指令不改变条件码，它用来计算地址的      
        + XOR，进位标志和溢出标志会设置成0      
        + 移位操作，进位标志将设置为 __最后一个被移出的位__，而溢出标志设置为0      
        + INC和DEC会设置溢出和零标志，不会改变进位标志          
19. 有两类指令只设置条件码而不改变其他任何寄存器        
    |指令|基于|描述|
    |-|-|-|
    |CMP S<sub>1</sub>, S<sub>2</sub>|S<sub>1</sub> - S<sub>2</sub>|比较|
    |cmpb||比较字节|
    |cmpw||比较字|
    |cmpl||比较双字|
    |cmpq||比较四字|
    |TEST S<sub>1</sub>, S<sub>2</sub>|S<sub>1</sub> & S<sub>2</sub>|测试|
    |testb||测试字节|
    |testw||测试字|
    |testl||测试双字|
    |testq||测试四字|

20. 条件码不会直接读取，常用的使用方法有三种：    
    + 根据条件码的某种组合，将 __一个字节__ 设置成0或者1，将这类指令成为 `SET` 指令       
    + 可以条件跳转到程序的某个其他部分      
    + 可以有条件的传送数据      
21. `SET` 指令的目的操作数是低位字节寄存器元素之一，或者是一个字节的内存位置，指令会将这个字节设置成0或者1.为了得到一个32位或者64位的结果。必须对高位清零。     
    |指令|同义名|效果|设置条件|
    |-|-|-|-|
    |sete D|setz|D &larr; ZF|相等/零|
    |setne D|setnz|D &larr; ~ZF|不相等/非零|
    |sets D||D &larr; SF|负数|
    |setns D||D &larr; ~SF|非负数|
    |setg D|setnle|D &larr; ~(SF^OF)&~ZF|大于（有符号>）|
    |setge D|setnl|D &larr; ~(SF^OF)|大于等于（有符号>=）|
    |setl D|setnge|D &larr; SF^OF|小于（有符号<）|
    |setle D|setng|D &larr; (SF^OF) \| ZF|小于等于（有符号<=）|
    |seta D|setnbe|D &larr; ~CF & ~ZF|超过（无符号>）|
    |setae D|setnb|D &larr; ~CF|超过或者相等（无符号>=）|
    |setb|setnae|D &larr; CF|低于（无符号<）|
    |setbe|setna|D &larr; CF \| ZF|低于或者相等（无符号<=）|

22. `jmp` 指令是无条件跳转，它有两种跳转方式：      
    + 直接跳转，跳转目标是作为指令的一部分编码的，汇编语言中，直接跳转是给出一个标号作为跳转目标的        
    + 简介跳转，跳转目标是从寄存器或者内存位置读出的，简介跳转的写法是 _*_  后面跟一个操作数指示符：      
      + `jmp *%rax`，用寄存器中的值作为跳转目标     
      + `jmp *(%rax)`，以 `%rax` 中的值作为读地址，从内存中读出跳转目标            

    |指令|同义名|跳转条件|描述|
    |-|-|-|-|
    |jmp Label||1|直接跳转|
    |jmp *Operand||1|间接跳转|
    |je Label|jz|ZF|相等/零|
    |jne Label|jnz|~ZF|不相等/非零|
    |js Label||SF|负数|
    |jns Label||~SF|非负数|
    |jg Label|jnle|~(SF^OF) & ~ZF|大于（有符号>）|
    |jge Label|jnl|~(SF^OF)|大于或者等于（有符号>=）|
    |jl Label|jnge|SF^OF|大小（有符号<）|
    |jle|jng|(SF^OF) \| ZF|小于或者等于（有符号<=）|
    |ja Label|jnbe|~CF & ~ZF|超过（无符号>）|
    |jae Label|jnb|~CF|超过或者相等（无符号>=）|
    |jb Label|jnae|CF|低于（无符号<）|
    |jbe Label|jna|CF \| ZF|低于或者相等（无符号<=）|

23. 跳转指令最常用的编码方式是PC相对的。它们会将目标指令的地址与紧跟在跳转指令后面那条指令的地址之间的差作为编码。这些地址偏移量可以编码为 __1，2或者4__ 个字节。第二种编码方法是给出“绝对地址”，用4个字节直接指定目标，汇编器和连接器狐疑选择适当的跳转目标编码。   
24. if-else       
    + 形式一      
      ```
      if (test-expr)
        then-statement
      else
        else-statement
      ```
    + 形式二，这种是通用的形式      
      ```
      t =test-expr
      if (!t)
        goto false;

      then-statement
      goto done;

      false:
        else-statement
      done:
      ```
    + 形式三        
      ```
      t = test-expr
      if (t)
        then-statement
      goto done;

      false:
        else-statement

      done:
      ```
    + 形式四，使用替换的策略是使用数据的跳转转移。这种方法是计算一个条件操作的两个结果，然后再根据条件是否满足从中选取一个。只有在一些受限制的情况中，这种策略才可行。但是如果可行的话，就可以用一条简单的条件传送指令来实现它，条件传送指令更符合现代处理器的性能特性。      
      ```
      v = then-expr
      ve = else-expr
      t = test-expr
      if (!t) v = ve
      ```
      |指令|同义名|传送条件|描述|
      |-|-|-|-|
      |comve S, R|cmovz|ZF|相等/零|
      |cmovne S, R|cmovnz|~ZF|不相等/非零|
      |cmovs S, R||SF|负数|
      |cmovns S, R||~SF|非负数|
      |cmovg S, R|cmovnle|~(SF^OF)&~ZF|大于（有符号>）|
      |cmovge S, R|cmovnl|~(SF^OF)|大于或者等于（有符号>=）|
      |cmovl S, R|comvnge|SF^OF|小于（有符号<）|
      |cmovle S, R|cmovng|(SF^OF) \| ZF|小于或者等于（有符号<=）|
      |cmova S, R|cmovnbe|~CF & ~ZF|超过（无符号>)|
      |cmovae S, R|cmovnb|~CF|超过或者相等（无符号>=）|
      |cmovb S, R|cmovnae|CF|低于（无符号<）|
      |cmovbe S, R|cmovna|CF \| ZF|低于或者等于（无符号<=）|

1.  过程(过程 P 调用过程 Q)      
    1.  一种抽象方式      
    2.  形式多样：函数，方法，子例程，处理函数等等      
    3.  要求：      
        1.  传递控制，PC 寄存器必须切换为 Q 过程的首地址      
        2.  传递数据，P 能够向 Q 传递多个参数，Q 能够返回给 P 一个值      
        3.  分配和释放内存，Q 有可能有临时变量，在执行完之后需要释放掉      
2.  ![alt 通用栈帧结构(栈用来传递参数/存储返回信息/保存寄存器/局部存储)](./pictures/通用栈帧结构.png "通用栈帧结构")      
3.  当 x86-64 过程需要的的存储空间超过存储寄存器能够存放的大小时，就会在栈上分配空间，这部分就被成为栈帧      
4.  当前执行的过程的栈帧总是在栈顶      
5.  P 调用 Q 时，返回地址：   
    1.  返回地址会被压入  __P__ 的栈中，因为它表示的时与 P 相关的状态     
    2.  当 Q 返回时，它表示要从 P 程序的哪个位置继续执行      
6.  Q 的代码会扩展当前栈的边界，分配它的栈帧所需的空间      
    1.  可以保存寄存器的值      
    2.  分配局部变量空间      
    3.  设置调用的参数      
7.  大多数过程的栈帧都是定长的，在过程的开始就分配好了。但是有些过程需要变长的帧。对于 x86-64 来说，参数个数小于等于6个时，那么所有的参数都可以通过寄存器传递，就不需要所谓的栈帧     
8.  将控制从 P 转移到 Q 只需要将 PC 寄存器设置为 Q 的起始位置：   
    |指令|描述|
    |-|-|
    |call label|过程调用|
    |call *operand|过程调用|
    |ret|从过程调用中返回|

    + call 指令有一个目标，即指明被调用过程起始的指令地址。     
      + 直接调用的目标时一个标号     
      + 间接调用的目标是 * 后面跟一个操作数指示符     
