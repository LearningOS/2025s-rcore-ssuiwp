### 实现功能

task::TaskManagerInner中添加syscall_counts字段，记录不同app调用syscall::syscall的次数，用syscall_id来作为查询index；

task::TaskManager中实现pub fn increment_syscall_count(&self, syscall_id: usize);pub fn get_syscall_count(&self, syscall_id: usize) -> isize ;用于增加计数和获取计数

按照要求实现syscall::process::sys_trace用于trace信息查询，
syscall::syscall中进入方法时调用静态示例task::TASK_MANAGER.increment_syscall_count(syscall_id)增加计数。

### 简答作业

1. 正确进入 U 态后，程序的特征还应有：使用 S 态特权指令，访问 S 态寄存器后会报错。 请同学们可以自行测试这些内容（运行 三个 bad 测例 (ch2b_bad_*.rs) ）， 描述程序出错行为，同时注意注明你使用的 sbi 及其版本。

   ```
   RustSBI version 0.3.0-alpha.2, adapting to RISC-V SBI v1.0.0
   RUSTSBI Implementation: RustSBI-QEMU Version 0.2.0-alpha.2
   ```

   报错

   ```
   [kernel] PageFault in application, bad addr = 0x0, bad instruction =    0x804003a4, kernel killed it.
   [kernel] IllegalInstruction in application, kernel killed it.
   [kernel] IllegalInstruction in application, kernel killed it.
   ```

   ```
   ch2b_bad_address.rs 内存地址0x0是无效的，实际是NULL，对该地址写入或引用会触发异常退出
   ch2b_bad_instructions.rs 在用户态非法使用指令sret
   ch2b_bad_register.rs 在用户态非法使用指令csrr
   ```
2. 深入理解 trap.S 中两个函数 __alltraps 和 __restore 的作用，并回答如下问题:

   1. L40：刚进入 __restore 时，sp 代表了什么值。请指出 __restore 的两种使用情景。

   ```
       刚进入 __restore 时，sp 指向内核栈，sp指向的值是TrapContext的起始地址。
       __restore 的作用包括:
       从系统调用或异常返回时, 通过restore恢复用户态的上下文信息，使程序能够继续在用户态执行。
       任务切换时, 内核保存当前任务的上下文，并通过__restore恢复要切换的任务的上下文信息，实现任务之间的切换
   ```

   2. L43-L48：这几行汇编代码特殊处理了哪些寄存器？这些寄存器的的值对于进入用户态有何意义？请分别解释。

   ```
   ld t0, 32*8(sp)
   ld t1, 33*8(sp)
   ld t2, 2*8(sp)
   csrw sstatus, t0
   csrw sepc, t1
   csrw sscratch, t2   
   ```

   ```
   ld t0, 32*8(sp)     # 内核栈 32*8(sp) 处存储了原 sstatus 寄存器的值, 将其读取到 t0
   ld t1, 33*8(sp)     # 内核栈 32*8(sp) 处存储了原 sepc 寄存器的值, 将其读取到 t1
   ld t2, 2*8(sp)      # 内核栈 2*8(sp) 处存储了原 sscratch 寄存器的值, 将其读取到 t2
   csrw sstatus, t0    # 将 t0中原 sstatus 寄存器的值读取到 sstatus
   csrw sepc, t1       # 将 t0中原 sepc 寄存器的值读取到 sepc
   csrw sscratch, t2   # 将 t0中原 sscratch 寄存器的值读取到 sscratch
   sstatus寄存器控制当前处理器的状态，恢復sstatus的值是为了确保返回用户态的时候，状态不会改变
   sepc寄存器存储了产生中断或异常前的指令地址，恢復sepc的值是为了确保返回用户态的时候，程序会继续从原来的指令地址开始执行
   sscratch寄存器用于保存用户栈指针，恢復sscratch的值是为了确保返回用户态的时候，能够正确的访问用户栈
   ```

   3. L50-L56：为何跳过了 x2 和 x4？

      x2是栈指针寄存器，跳过x2是因为在__alltraps中x2对应的用户栈指针已经保存到了sscratch寄存器，所以不需要从内核栈中进行恢复。
      x4是线程指针寄存器，用于多线程环境的线程本地存储，跳过x4是因为在当前环境中，未使用x4。
   4. L60：该指令之后，sp 和 sscratch 中的值分别有什么意义？

      交换sp和sscratch的值，sp指向用户栈，sscratch指向内核栈
   5. __restore：中发生状态切换在哪一条指令？为何该指令执行之后会进入用户态？

      状态切换发生在sret后，执行该指令后, 完成从监督态到用户态的切换。将sepc的值加载到PC（程序计数器）中，sepc 存储着产生中断或异常前的指令地址，使程序从用户态被中断的地方继续执行。
   6. L13：该指令之后，sp 和 sscratch 中的值分别有什么意义？

      csrrw sp, sscratch, sp

   ```
   交换sp和sscratch的值，sp指向内核栈，sscratch指向用户栈。 
   ```

   7. 从 U 态进入 S 态是哪一条指令发生的？

      ecall指令，它是用户态程序向操作系统内核发起请求的主要方式。ecall触发一个类型为Environment call from U-mode to S-mode 的中断。ecall触发后，处理器从用户态切换到监督态，并将控制权交给内核的异常处理程序。

### 荣誉准则

1. 在完成本次实验的过程（含此前学习的过程）中，我曾分别与 以下各位 就（与本次实验相关的）以下方面做过交流，还在代码中对应的位置以注释形式记录了具体的交流对象及内容：

   无

   2. 此外，我也参考了 以下资料 ，还在代码中对应的位置以注释形式记录了具体的参考来源及内容：

      (https://learningos.cn/rCore-Tutorial-Guide-2025S/chapter3/5exercise.html)
      (https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/5exercise.html)

      3. 我独立完成了本次实验除以上方面之外的所有工作，包括代码与文档。 我清楚地知道，从以上方面获得的信息在一定程度上降低了实验难度，可能会影响起评分。
      4. 我从未使用过他人的代码，不管是原封不动地复制，还是经过了某些等价转换。 我未曾也不会向他人（含此后各届同学）复制或公开我的实验代码，我有义务妥善保管好它们。 我提交至本实验的评测系统的代码，均无意于破坏或妨碍任何计算机系统的正常运转。 我清楚地知道，以上情况均为本课程纪律所禁止，若违反，对应的实验成绩将按“-100”分计。
