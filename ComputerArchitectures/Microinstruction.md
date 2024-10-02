## XOR R0, R0

```vhdl
R0 OUT, Y IN, ALU XOR; --Send R0 both to Y and ALU, do the XOR operation, store the result to Z
Z OUT, R0 IN; --Copy the content of Z back to R0
```



## SUB R0, R1

```vhdl
R0 OUT, Y IN; --Copy R0 to the temporary register Y
R1 OUT, ALU SUB; --Send R1 to ALU, do the SUB operation, store the result to Z
Z OUT, R0 IN; --Copy the content of Z back to R0
```



## SUB R0, [R1]

```vhdl
R1 OUT, MAR IN; --Copy the address in R1 to MAR
MEMORY READ, R0 OUT, Y IN; --Copy R0 to Y while reading from memory
MDR OUT, ALU SUB; --The data retrived from the memory has been copied to MDR, sent it to the ALU, do SUB operation, store the result to Z
Z OUT, R0 IN; --Copy the content of Z back to R0
```



## DEC R1

```vhdl
R1 OUT, ALU DEC; --Send R1 to ALU, do the SUB operation, store the result to Z
Z OUT, R1 IN; --Copy the content of Z back to R1
```



## NOT [R1]

```vhdl
R1 OUT, MAR IN; --Copy the address in R1 to MAR
MEMORY READ; --There is a delay of time to retrieve the data from the memory
MDR OUT, ALU NOT; --The data retrieved from the memory has been copied to MDR, send it to the ALU, do NOT operation, store the result to Z
Z OUT, MDR IN; --Copy the content of Z to MDR
MEMORY WRITE; --There is a delay of time to write the data back to the memory
```



## INC [R1]

```vhdl
R1 OUT, MAR IN; --Copy the address in R1 to MAR
MEMORY READ; --There is a delay of time to retrieve the data from the memory
MDR OUT, ALU INC; --The data retrieved from the memory has been copied to MDR, send it to the ALU, do INC operation, store the result to Z
Z OUT, MDR IN; --Copy the content of Z to MDR
MEMORY WRITE; --There is a delay of time to write the data back to the memory
```



## SUB [R1], 5

```vhdl
R1 OUT, MAR IN; --Copy the address in R1 to MAR
MEMORY READ, IR OUT, Y IN; --Copy the constant 5 from IR to Y while reading from memory
MDR OUT, ALU SUB; --The data retrieved from the memory has been copied to MDR, send it to the ALU, do SUB operation, store the result to Z
Z OUT, MDR IN; --Copy the content of Z to MDR
MEMORY WRITE; --There is a delay of time to write the data back to the memory
```



## MOV [R1], 0 🌟

```vhdl
R1 OUT, MAR IN; --Copy the address in R1 to MAR
-- It doesn't require memory read
IR OUT, MDR IN; --Copy the constant 0 from IR to MDR
MEMORY WRITE; --There is a delay of time to write the data back to the memory
```

