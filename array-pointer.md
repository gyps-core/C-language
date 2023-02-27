# 配列とポインタ
配列とポインタの違いについて、扱っていてどのような差異が生まれるのか、あまり具体的に言語化できなかったのでまとめてみる。  
### 1, 配列の実態
ポインタと配列の違いがよくわからなくなる理由として、配列の扱い方があると思います。
まず基本的な配列についてみていきましょう。
初期化に関しては以下のように取り扱うことができ,型のことはint型ではなくint[]型と読みます。  
```c
//初期化
int array[];
int array2[4];
int array3[4]={0};
```
ややこしくなるのは実際に配列を扱うときです。
```c
//実数値を扱える
array[1] = 15;
int x = array[2];
//アドレスとしても扱える
int *x2 = array;
int *x3 = &array[2];
```
### 1, 根本的な違い
配列とポインタについての違いをアセンブリレベルで確認する。
以下に配列を用いた初期化関数とポインタを用いた初期化関数のC言語とMips gcc 12.2.0でコンパイルしたアセンブリを記述する。
```c
// 配列を用いた初期化
void clear_array(int array[], int size){
  int i;
  for(i=0; i<size; i+=1){
    array[i]=0;
  }
}
```
```s
clear_array:
        addiu   $sp,$sp,-24
        sw      $fp,20($sp)
        move    $fp,$sp
        sw      $4,24($fp)
        sw      $5,28($fp)
        sw      $0,8($fp)
        b       $L2
        nop

$L3:
        lw      $2,8($fp)
        nop
        sll     $2,$2,2
        lw      $3,24($fp)
        nop
        addu    $2,$3,$2
        sw      $0,0($2)
        lw      $2,8($fp)
        nop
        addiu   $2,$2,1
        sw      $2,8($fp)
$L2:
        lw      $3,8($fp)
        lw      $2,28($fp)
        nop
        slt     $2,$3,$2
        bne     $2,$0,$L3
        nop

        nop
        nop
        move    $sp,$fp
        lw      $fp,20($sp)
        addiu   $sp,$sp,24
        jr      $31
        nop
```

```c
// ポインタを用いた初期化
void clear_pointer(int *array, int size){
  int *p;
  for(p=&array[0];p<&array[size];p=p+1)
    *p = 0;
}

```

```s
clear_pointer:
        addiu   $sp,$sp,-24
        sw      $fp,20($sp)
        move    $fp,$sp
        sw      $4,24($fp)
        sw      $5,28($fp)
        lw      $2,24($fp)
        nop
        sw      $2,8($fp)
        b       $L2
        nop

$L3:
        lw      $2,8($fp)
        nop
        sw      $0,0($2)
        lw      $2,8($fp)
        nop
        addiu   $2,$2,4
        sw      $2,8($fp)
$L2:
        lw      $2,28($fp)
        nop
        sll     $2,$2,2
        lw      $3,24($fp)
        nop
        addu    $2,$3,$2
        lw      $3,8($fp)
        nop
        sltu    $2,$3,$2
        bne     $2,$0,$L3
        nop

        nop
        nop
        move    $sp,$fp
        lw      $fp,20($sp)
        addiu   $sp,$sp,24
        jr      $31
        nop
```
