---
title: "VHDL: Synthesis"
categories:
  - Digital design
tags:
  - VHDL
  - Digital
---

# VHDL / synthesis
Created Wednesday 30 January 2019


VHDL File Structure
-------------------

### File name
``entity-name_architecture-name.vhd``

### Standard header

```VHDL
--*****************************************************************************
-- Company			EPFL-LSM
-- Project Name		Medusa
--*****************************************************************************
-- Doxygen labels
--! @file 			#optional file name#
--! @brief 			a short description what can be found in the file
--! @details 		detailed description
--! @author 		Selman Ergünay
--! @email 			selmanerg@gmail.com
--! @date 			2016-03-30
--*****************************************************************************
-- Revision History	:
--   Rev 0.0 - File created.
-- TODO:
--*****************************************************************************
-- Naming Conventions:
--   active low signals:                    "*_n"
--   clock signals:                         "clk", "clk_div#", "clk_#x"
--   reset signals:                         "rst", "rst_n"
--   generics:                              "C_* -all UPPERCASE"
--   state machine current/next state:      "*_cs" / "*_ns"         
--   pipelined or register delay signals:   "*_d#"
--   counter signals:                       "*cnt*"
--   data valid signals			    "*_vld"
--   internal version of output port:       "*_i"
--   ports:                                 "- Names begin with Uppercase"
--   processes:                             "*_PROC"
--   component instantiations:              "<ENTITY_>I_<#|FUNC>"
--*****************************************************************************
```

### Libraries

<http://standards.ieee.org/downloads/1076/1076.2-1996/std_logic_1164.vhdl>

<http://standards.ieee.org/downloads/1076/1076.2-1996/std_logic_1164-body.vhdl>

<https://standards.ieee.org/downloads/1076/1076.2-1996/numeric_std.vhdl>

```VHDL
library ieee;
use ieee.std_logic_1164.all;	-- std_logic, logical operations, rising_edge, falling_edge	
use ieee.std_numeric.all;	-- signed and unsigned type, arithmetic operations	
use textio.all;
```

Predefined logical design libraries:

    ``work``	: active working library
    ``std`` 	: standard, textio
    ``ieee``	: std_logic


### Entity declaration

```VHDL
entity serg is 
  generic
  port(
    clk	: in std_logic;
    rst	: in std_logic;

    sum	: out std_logic);
end entity serg;
```


* Entity'de bulunan tüm sinyallerin bit sayılarını belirten değişkenler generic'te bulunmalı.
  

### Logic communication representation
```VHDL
	signal clk : std_logic;
	signal clk : std_logic := '0';
	
	signal data_bus : std_logic_vector(31 downto 0);
	signal data_bus : std_logic_vector(31 downto 0) := (others => '0');
	signal data_bus : std_logic_vector(31 downto 0) := "0000_0000";	
	
	signal count 	: integer := 0;
	--Naming:
	signal signalname_s : signed(7 downto 0);
	signal signalname_u	: unsigned(7 downto 0);
```

### Logic data representation

```vhdl
	type std_ulogic is ( 'U', -- Uninitialized
						 'X', -- Forcing Unknown
						 '0', -- Forcing 0
						 '1', -- Forcing 1
						 'Z', -- High Impedance
						 'W', -- Weak Unknown
						 'L', -- Weak 0
						 'H', -- Weak 1
						 '-' -- Don't Care
						);
	type std_logic_vector is array (natural range <>) of std_logic;
```

### Type declaration

```vhdl
	type <type_name> is array integer range <lower_limit> to <upper_limit>;	--array of integers
	type <type_name> is (<string1>, <string2>, ...); 			--enumeration
	type <type_name> is type <type_definition>; 				--general
```

Operators
---------

### Relational / comparison operators

    `=` 	`/=`	`<`	`<=`	`>`	`>=`


* interpreter'da normalde slv karsilastirma tanimli olmamasi lazim, cunku slv kavrami std_logic_1164 ile geliyor.
* Bu yuzden "" arasindaki herhangi bir seyi string olarak alip karsilastiriyor.
* Note: Although they are not defined in std_logic_1164, XST assumes the operands are unsigned and compute.
* defaultta unsigned karsilastirma yapiyor sentezleyici. bence
* It shows that slv comparison is pre-built function resolved by synthesizer.


These operations are overwritten by `numeric_std`.

Definition in `numeric_std`:

```vhdl
	function SIGNED_equal ( L,R: SIGNED) return BOOLEAN is
		begin
		return STD_LOGIC_VECTOR (L) = STD_LOGIC_VECTOR (R) ;
	end;
```


* For comparison operators the result is boolean.


### Logical operators

    `and` 	`or`	`nand` 	`nor`	`xor`	`xnor`

and ile ya bool ifade olacak, ya da sinyal. ikisi karışık olmuyor.

### Shift and rotate

    `sll`		`srl`		`rol` 	`ror`

### Concatenation:

    `&`

<https://stackoverflow.com/questions/28634875/ambiguous-type-in-infix-expression-vhdl>

Concat yaparken her sey std_logic_vector olsun. Atama yapılan sinyal de.

The problem sometimes with arrays of vectors is that the compiler doesn't know if you intend to concatenate two vectors into a single vector, or two vectors into a 2-vector array. You need to tell it what you're intending to concatenate as:


Dataflow architecture
---------------------

```vhdl
	architecture dfl of serg is	
			-- declarative part
	begin 	-- of statement part (only concurrent statements)
	
	--Concurrent signal assignment statement:
		P1 	: sum 	<= opa xor opb xor cin;
		P2	: cout	<= (opa and opb) or (opa and cin);
	end architecture dfl;
```

Equivalent concurrent process statements:

```vhdl
	P2: process(opa, opb, cin)
	begin 
		cout	<= (opa and opb) or (opa and cin);
	end process P1;
```

Signal assignment
-----------------

A signal object is stored in a specific data structure called a driver whose role is to maintain the signal's history.

Transactions that will lead to a change of the current value are called events.


* Sanki simulasyon ile buyuyen bir linked list gibi. Dolayisiyla simulasyonu yavas, bellek istiyor. Fakat sinyaldeki degisimleri tuttugu icin takip edilebilir.



* Variable ise normal C degiskeni gibi. Degisimler takip edilemiyor normalde.

```vhdl
	sum 	<= opa xor opb xor cin;
	sum 	<= opa xor opb xor cin after 2 ns;			-- after clause is ignored in synthesis
	sum 	<= '0', '1' after 5 ns, '0' after 15 ns;	-- after clause is ignored in synthesis
```


```vhdl
signal A_uv, B_uv, C_uv, D_uv, E_uv : unsigned(7 downto 0) ;
signal R_sv, S_sv, T_sv, U_sv, V_sv : signed(7 downto 0) ;
signal J_slv, K_slv, L_slv : std_logic_vector(7 downto 0) ;
signal Y_sv : signed(8 downto 0) ;
```

Permitted:
```vhdl
A_uv 	<= B_uv + C_uv ; 	-- Unsigned + Unsigned 	= Unsigned
D_uv 	<= B_uv + 1 ; 		-- Unsigned + Integer 	= Unsigned
E_uv 	<= 1 + C_uv; 		-- Integer 	+ Unsigned 	= Unsigned
R_sv 	<= S_sv + T_sv ; 	-- Signed 	+ Signed 	= Signed
U_sv 	<= S_sv + 1 ; 		-- Signed 	+ Integer 	= Signed
V_sv 	<= 1 + T_sv; 		-- Integer 	+ Signed 	= Signed
J_slv 	<= K_slv + L_slv ; 	-- if using std_logic_unsigned
```

### Strong Typing Implications

VHDL is strongly typed so it won't let you connect incorrect types without converting them properly.

```vhdl
--Operation 		--Size of Y = Size of Expression
Y <= "10101010"; 	--number of digits in literal
Y <= X"AA"; 		--4 * (number of digits)
Y <= A; 		--A'Length = Length of array A
Y <= A and B; 		--A'Length = B'Length
W <= A > B ; 		--Boolean
Y <= A + B ; 		--Maximum (A'Length, B'Length)
Y <= A + 10 ; 		--A'Length
V <= A * B; 		--A'Length + B'Length
```


* Mesela `A <= 0;` diye bir atama olmuyor, cunku sag tarafin kac bit oldugu belli degil. Bu belirsizlik vhdl amcanin hosuna gitmiyor. 0 burada integer. Atama dedigin bit boyutunda yapiliyor. Karsilastirmalar falan olabilir onlar ayri. Ama atama bit bit olacak.


### Assignment 

```vhdl
signal A8, B8, Result8 : unsigned(7 downto 0) ;
signal Result9 : unsigned(8 downto 0) ;
signal Result7 : unsigned(6 downto 0) ;

-- Simple Addition, no carry out
Result8 <= A8 + B8 ;
-- Carry Out in result
Result9 <= ('0' & A8) + ('0' & B8) ;
-- For smaller result, slice input arrays
Result7 <= A8(6 downto 0) + B8(6 downto 0) ;
```

```vhdl
function "+" (L,R: UNSIGNED ) return UNSIGNED;
 -- Result subtype: UNSIGNED(MAX(L'LENGTH, R'LENGTH)-1 downto 0).
 -- Result: Adds two UNSIGNED vectors that may be of different lengths.
```


* Bu demek oluyor ki toplamin bit sayisi toplanan sayilardan bit sayisi en buyuk olani ile ayni.


<a name="process"/>

## Process statement
Process simulation life cycle:

* **Created**    at elaboration with its local declarations
* **Activated**	when at least one signal in the sensitivity list has a change value::event
* **Stopped**     once its statements have been executed
* **Destroyed**	when simulation finishes 

```vhdl
P1	: process(a, b, c) is
--Declarative part: only variables can be declared
begin
-- sequential statement part
d <= a xor b;
end process P1;
```

Process activation control on signal events: 

```vhdl
process(signal_name_list) 	--option 1: sensitivity list forbids wait statement, or vice versa
process  					--option 2: equivalent statement
wait on signal_name_list 
--on timeout: 
wait for 10 ns;
--on condition: 
wait until enable = '1' for 1 ms; 	-- whichever comes first

wait; 	-- stop forever
```

Example of a stimulus generation process:

```vhdl
signal s: std_logic := '0';
STIM_GEN1 : process
begin
wait for 5 ns;
s 	<= '1', '0' after 10 ns, '1' after 18 ns;
wait;
end process STIM_GEN1;
STIM_GEN2 : process
begin
wait for 5 ns;
s <= '1';
wait for 10 ns;
s <= '0';
wait for 8 ns;
s <= '1';
wait;
end process STIM_GEN2;
```

### Process simulation model

Initialization phase:

* **I1:** Each signal and variable gets a default  or defined initial value.
* **I2:** Current simulation time Tc:=0.
* **I3:** [Process execution phase] Each process is executed until it suspends.
* **I4:** Computation of the next simulation time Tn, which is the earliest of:


* next time of a pending transaction
* next time of end of timeout (wait for)
* value time'high
* if Tn = Tc, the next simulation cycle is delta cycle.



Simulation phase:

**S1:** Tc := Tn.

**S2:** [Signal update phase] Current values of signals get their new values.

**S3:** [Process execution phase] Process sensitive to signal changes are executed.

**S4:** Tn = next simulation time as in I4


* If Tn = time'high and no more pending transactions, simulation stops.
* Otherwise go to S1.


## Algorithmic architecture

Sequential statements always execute in order of their appearance, while concurrent statements may execute in any order.

Variables are another class of VHDL objects that can only be declared and used inside a process.

Variable assignment statement: `:=`

```vhdl
architecture algo of fadd1 is
  use ieee.numeric_std.all;
begin
  process(opa, opb, cin)
    variable tmp : unsigned(1 downto 0);
  begin
    tmp := (others => '0');
    if opa = '1' then tmp := tmp+1; end if;
    if opb = '1' then tmp := tmp+1; end if;
    if cin = '1' then tmp := tmp+1; end if;
    if tmp > 1 then
      cout <= '1';
    else	
      cout <= '0';
    end if;
    if tmp mod 2 = 0 then
      sum <= '0';
    else 
      sum <= '1';
    end if;
  end process;
end architecture algo;
```


### Signal vs variable

	| Signal                              | Variable                      |
	|-------------------------------------|-------------------------------|
	| Represent HW signals                | Represent temporary storage   |
	| Global scope (design entity)        | Local scope (process)         |
	| Complex data structure              | Memory location only          |
	| Assignment with <=                  | Assignment with :=            |
	| Updated one step after all          | Updated immediately after the |
	| processes have finished executing   | assignment                    |
	| Most appropriate for concurrent bhv | Algorithmic bhv               |
	| Can be tracked                      | CANNOT be tracked             |


* When modeling memory (RAM), it is better to use a variable.



* Because variables do not have an event queue associated, their space requirement (memory footprint on your workstation/PC) is about ten times smaller than the equivalent signal. 



* For large RAMs this could make the difference between being able to run, or just thrashing your disk caused by swapping.



*****


### Loops


* Concurrent loop generate statement



* Preprocessing macro statement:



* Process won't exist anymore when the model is executed.



* Cannot be used in process.


```vhdl
STAGES 	: for k in 0 to 3 generate
BS	: s(k)		<= a(k) xor b(k) xor ci(k);
BCO	: ci(k+1)	(a(k) and b(k));
end generate STAGES;
```

Unrolled stages: STAGES(0) ... STAGES(3) 


* Generic n-bit adder using components:


```vhdl
STAGES: for k in NBITS-1 downto 0 generate
signal s_unbuffered: std_logic;
begin
LSB : if k = 0 generate
BIT: component c_fadd1
port map (a => opa(0), b => opb(0), ci => '0');
end generate LSB;
OTHERB : if k /= 0 generate
BIT: component c_fadd1
port map (a => opa(k), b => opb(k), ci => ci(k));
end generate OTHERB;
end generate STAGES;
```

```vhdl
NOCLK200_CHECK : if ( NOCLK200 = false ) generate
begin
  IDL_CLK_INST : IBUFG
  port map(
    I  => idly_clk_200,
    O  => clk200_in
  );
end generate;
```


### Sequential for loop statement


* The synthesis interpretation: replicate the hardware 


```vhdl
process(a, tmp)
begin
tmp(0) <= a(0);
for i in 1 to 3 loop 		--Loop index does not need to be declared. Only visible in the loop body.
tmp(i) <= a(i) xor tmp(i-1);
end loop;
end process;
```


* Generic parameter


```vhdl
entity serg is 
generic();
port();
end entity serg;
```

```vhdl
entity serg is 
generic( constant NBITS : positive := 8); 	--integer can be used as well, but positive makes sense.
port();
end entity serg;
```

<a name="comb"/>

## From VHDL models to combinational circuits

### Simple concurrent signal assignment:


* Delay clauses are ignored.


```vhdl
sum <= opa and opb after 2ns;
```


* Multiple element waveforms are not allowed.


```vhdl
s <= '1', '0' after 10 ns;
```


* Supported signal assignment:


```vhdl
signal_name <= value;
```


* Avoid zero-delay feedback loops


```vhdl
a <= a and b;
```


* Input signals may be only read, output ports may be only written.



* Avoid multiple concurrent signal assignment onto the same signal.


```vhdl
O1 :	z <= a or c;
A1 : 	z <= a and b;
```


* signal `z` may get the value `X` in the simulation.



* `std_logic` and `std_logic_vector` are called resolved types.


  * signals of these types may have more than one source.

  * their actual values at a given time is determined by using a resolution algorithm.

  * this algorithm is always executed for std_logic(_vector) even if they have 1 source.


* ieee.std_logic_1164 also defines unresolved types: 


```vhdl
signal a: std_ulogic 
signal b: std_ulogic_vector(7 downto 0);
```

allows the detection of unwanted multiple source signals at analysis or elaboration step.

above example: syntactically correct, but hw lead incorrect design.

using unresolved types make the analysis fail.

A resolution function defines how values from multiple sources, multiple drivers, are resolved into a single value.

resolution table

### Conditional / selected signal assignments


* Condition bool bir ifade olmali. -- TODO :: test



* Concurrent conditional signal assignment -- more general wrt with, can be any condition.



* birden fazla kosul yazma imkani oldugunda kosullarin bittigini soylemek icin bir keyword gerekli. 

o yuzden if'de then, bunda else var.

	z 	<= 	a when sel = "00" else
			b when sel = "01" else
			c when sel = "10" else
			d;



### Concurrent selected signal assignment

	with tti select
		tto <= 	"00" when "000",
				"01" when "110" ¦ "011" ¦ "101",
				"11" when "111",
				"XX" when others;




* When others clause is necessary because std_logic have 9 values.


### Combinational Process with Case Statement

	process(a)
	begin
		case a is
			when "00" => b <= "1000";
			when "01" => b <= "0100";
			when "10" => b <= "0010";
			when "11" => b <= "0001";
			when others => report "unreachable" severity failure;
		end case;
	end process;


### Combinational logic from a process statement

A process expresses a combinational behaviour if and only if:


1. The process has a sensitivity list.
2. All signals that are read in the process are in the sensitivity list.
3. The process does not declare variables or, if it does, they are assigned before they are read.
4. All variables or signals are assigned in every branch of the execution flow in the process.


<a name="seq"/>

Sequential assignments
----------------------

### Simple sequential signal assignment

If multiple signal assignments to the same signal, only last active one counts:

	process(a,b,c)
	begin
		z <= a and b;
		z <= a or c;
		z <= b xor c;  	-- this counts
	end process;




### Sequential if statement

Unlike concurrent conditional signal assignment, any number of sequential assignments in any branch.

	process(a, b, c, sel)
	begin
		if sel = "00" then
			z <= a;
		elsif sel = "01" then 
			z <= b;
		else
			z <= c;
		end if;
	end process;




### Priority

	process(a, b, c, sel)
	begin
		z <= '0'; 	--to guarantee the combinational behaviour
		if sel(0) = '1' then 
			z <= a;
		end if;
		if sel(0) = '1' then 
			z <= b;
		end if;
		if sel(0) = '1' then 
			z <= c;
		end if;
	end process;



* Last if statement has the highest priority



* Avoid different signal targets:


	process(a,b,sel)
	begin
		if sel = '1' then 
			z <= a;
		else 
			y <= b;
		end if;
	end process;


### Sequential case statement

	architecture tt_case of fadd1 is
		signal tti : std_logic_vector(2 downto 0);
		signal tto : std_logic_vector(1 downto 0);
	begin
	tti <= a & b & ci;
	process(tti)
	begin
		case tti is 
			when "000" => 
				tto <= "00";
			when "110" ¦ "011" ¦ "101" => 
				tto <= "01";
			when "010" ¦ "100" ¦ "001" => 
				tto <= "11";
			when others => 
				tto <= "XX";
		end case;
	end process;
	s 	<= tto(1);
	co 	<= tto(0);
	end architecture tt_case;



* This architecture has four concurrent process. Three processes are simple concurrent signal assignments.


```vhdl
case a is 
when "" => ;
when "" => ;
when others =>;
end case;
```

### Variable assignment statement

	process(a)
		variable tmp: std_logic;
	begin
		tmp := a(0);
		for i in 1 to 3 loop 		--Loop index does not need to be declared. Only visible in the loop body.
			tmp := a(i) xor tmp;
		end loop;
		z <= tmp;
	end process;



* Equivalent:


	process(a)
	begin
		z <= a(0) xor a(1) xor a(2) xor a(3);
	end process;



### 'Z' interpretation

```vhdl
process(en,d)
begin
q <= 'Z';
if en = '1' then 
q <= d;
end if;
end process;
```

```vhdl
process(en,d)
begin
if en = '1' then 
q <= d;
else 
q <= 'Z';
end if;
end process;
```

### Arithmetic operations

numeric_std


* no multiple operation


### Multiple sources and three-state buffer


* Multiple buffers that are connected to the same output must be described using seperate conncurrent statemenets.



* i.e. either concurrent signal assignment or equivalent processes.


```vhdl
architecture dflz of multiple2 is
begin 
O1: z <= a or c when en1 = '1' else 'Z';
A1: z <= a and b when en2 = '1' else 'Z';
A2: z <= c and d when en3 = '1' else 'Z';
end architecture dflz;
```


### Bidirectional three-state buffer


* I/O pads and bus interfaces


```vhdl
entity buf3s is
port(
signal sout: in std_logic_vector(7 downto 0);
signal sin : out std_logic_vector(7 downto 0);
signal dir : in std_logic;
signal sbi : inout std_logic_vector(7 downto 0)
);
end entity buf3s;
architecture dual of buf3s is
begin 
sbi <= sout when dir = '1' else (others => 'Z');
sin <= sbi when dir = '0' else (others => 'Z');
end architecture dual;
```

```vhdl
type unsigned is array (natural range <>) of std_logic;
type signed is array (natural range <>) of std_logic;
signal a_u: unsigned(7 downto 0);
signal a_s: signed(7 downto 0);
use ieee.numeric_std.all;


* Working with types std_(u)logic(_vector) and (un)signed



* Type casting does not infer any hardware functionality.


```vhdl
C_sulv	<= std_ulogic_vector(D_slv);
C_sulv	<= std_logic_vector(D_slv);
E_uv 	<= signed(D_slv);
F_sv 	<= signed(D_slv);

D_slv 	<= std_logic_vector(unsigned(D2_slv) + unsigned(D3_slv));
F2_sv 	<= signed('0' & C2_sulv) - signed('0' & C3_sulv);
```

### Resize functions

z 		<= resize(a, z'length);


* Instead of this, simple slicing is more clear, especially for signed signals.


### Conversion functions:

	G_int	<= to_integer(E_uv);
	H_nat	<= to_integer(E_uv);
	
	G_int	<= to_integer(F_sv);
	
	E_uv	<= to_unsigned(G_int, E_uv'length);
	F_sv	<= to_signed(G_int, F_sv'length);
	
	E_uv 	<= to_unsigned((G2_int + G3_int), E_uv'length);
	G_int 	<= to_integer(F2_sv + F3_sv);


### Hardware realization of operators

From the HIGHEST priority to the least:


1. `**`, `abs`, `not`
2. *, /, mod, rem
3. +, - (unary versions)
4. +, -, &
5. sll, srl, sla, sra, rol, ror
6. `=`, `/=`, `<`, `<=`, `>`, `>=`, `?=`, `?/=`, `?<`, `?<=`, `?>`, `?>=`
7. and, or, nand, nor, xor, xnor



* Shift rotate: `sll` `srl` `sla` `sra` `rol` `ror` : not supported for synthesis. Instead use `shift_left`, `shift_right`, `rotate_left`, `rotate_rigt`  at numeric_std



* Shift & Rotate bit rearrangements:

  * When distance is constant: bit rearrangement is hardwired.
  * When distance is non-constant: a combinational barrel shifter is inferred.

* `/` `mod` `rem`: not recommended in synthesis



* `/` `*` if the operands are power of 2, infers a simple r/l shift

 	tested: xilinx does not place a multiplier for a*2

### Component declaration and component instantiation

* Component declaration defines what is needed,
* Entity decleration defines what is available.
* A configuration may be required to bind both.


#### Component declaration:

```vhdl
component component_name is
generic();
port();
end component component_name;
```

#### Component instantiation

in an architecture statement part:

```vhdl
lbl: component component_name
generic map(NBITS => 32)
port map();
```


* Port map'de sag tarafta bir islem (&, and, or vb) olamaz. Sadece sinyal olabilir.



* Direct instantiation: (no need component declaration)



* Library (work) must be added.


```vhdl
lbl: entity libname.entityname(architecturename)
port map();
--Here, there is no global bindings of the component instances to any particular design entities
--(entity-architecture pairs)

*****

--Configuration declaration
--Defines the top level of a hw model.
--In direct instantiation, configuration no more needed.

*****

--Arrays and records
--Array:
type array_type1 is array (0 to 3) of std_logic_vector(11 downto 0);  --first define the type of array.
signal array_name1 : array_type1;  --array_name1 is a 4 element array of integers.
--an example
signal test1 : std_logic_vector(11 downto 0);
test1 <= array_type2(0);
signal test2 : integer;
test2 <= array_type1(2);
--Record:
type record_name is
  record
 a : std_logic_vector(11 downto 0);
 b: std_logic_vector(2 downto 0);
 c : std_logic;
  end record;
type array_type3 is array (0 to 3) of record_name; --first define the type of array.
signal actual_name : array_type3;
--accessing the record.
a1 : std_logic_vector(11 downto 0);
b1: std_logic_vector(2 downto 0);
c1 : std_logic;
a1 <= actual_name(1).a;
b1 <= actual_name(1).b;
c1 <= actual_name(1).c;
actual_name(2).a <= "100011100011";
actual_name(1) <= (a =>  "100011100011", b => "101", c => '1');
actual_name(0) <= ("100011100011","101",'1');
```

### Describing sequential/synchronous behaviour


* Edge sensitive (flip-flop)


```vhdl
signal clk : std_logic;
process(clk)
begin 
if rising_edge(clk) then	--falling_edge(clk)
--clock-driven behaviour
end if;
end process;
```


* Equivalent:


```vhdl
process
begin
wait until clk = '1';	-- or wait until rising_edge(clk);
end process;
```


* Latch: not preferred for synchronous design


```vhdl
process(en, d)
begin 
if en = '1' then
q <= d;--transparent-latch behaviour
end if;
end process;
```



* Avoid latches


```vhdl
architecture latched of E3 is
signal C,D : std_logic;
begin
process(en, A, B, C, D)
begin
if en = '1' then
C <= A xor B;
D <= A or C;
Z <= C nand D;
end if;
end process;
end architecture latched;
```

```vhdl
architecture comb of E3 is
signal C,D : std_logic;
begin
process(en, A, B, C, D)
begin
C <= '0';
D <= '0';
Z <= '0';
if en = '1' then
C <= A xor B;
D <= A or C;
Z <= C nand D;
end if;
end process;
end architecture comb;
```

# SYNCHRONOUS DESIGN


* Clock and asynchronous reset signals never participate in logic operations (exception: clock gating, but only if you know what you are doing)



* Data/control signals never trigger a state transition



* All storage elements do have an asynchronous reset



* Do not use clock gating unless really needed and if so adhere to its special rules



* Use only a single clock in the entire circuit; if you need multiple clocks, keep them synchronized and use frequencies that are integer multiples of each other.




## Describing reset for edge-sensitive behaviour


* The initialization of signals shall be always described using a (a)synchronous reset (or set) signal. 
* The specification of a default value in a signal/variable declaration, although legal in VHDL, is ignored in synthesis.


### Synchronous reset:
+ Easy to synthesize
+ Just another clocked signal
+ Glitch-free (at gate input)
+ Small standard cell area
+ Easy static timing analysis
+ Can be safely used when circuit is working
- Requires a free-running clock
- Behaviour depends on clock speed

```vhdl
signal clk, rst: std_logic;
process(clk)
begin 
wait until rising_edge(clk);
if rst = '1' then 
--initializations
else
--clock-driven behaviour
end if;
end process;
```

### Asynchronous reset
+ Clock independent
+ More efficient power-up behaviour
+ Std cells usually have built-in async. sets/resets
+ Can be safely used for initialization
- Large standard cell area
- Not glitch free
- May require specific buffering and routing (reset tree)
- Must be synchronously deasserted

```vhdl
process(clk, rst)
begin 
if rst = '1' then 
--initializations
elsif rising_edge(clk) then
--clock-driven behaviour
end if;
end process;
```

### Template for synchronous circuits

	architecture rtl of sync_circuit is
		signal state_reg, state_next 	: reg_type;
		signal other_reg, other_next 	: other_type;
	begin
		-- memory elements (registers)
		REG: process(clk, rst)
		begin
			if rst = '1' then
				state_reg 	<= ...;
				other_reg 	<= ...;
			elsif clk'event and clk = '1' then
				state_reg 	<= state_next;
				other_reg 	<= other_next;
			end if;
		end process REG;
		-- next-state logic
		NSL: process(list_of_read_signals)
		begin
			state_next	<= F1(state_reg, ..._reg, inputs);
			other_next 	<= F2(..._reg, inputs);
		end process NSL;
		-- output logic
		OL: process(list_of_read_signals)
		begin
			outputs 	<= F3(..._reg, inputs);
		end process;
	end architecture rtl;	



--Using a single process for describing the counter's RTL behaviour may be more compact, but:
--	- Since the whole behaviour is clocked, more registers than necessary can be inferred.

### 4-bit binary counter

	entity bin_counter_4b is
		port(
			signal clk, rst		: in std_logic;
			signal clr, load, en: in std_logic;
			signal din 			: in std_logic_vector(3 downto 0);
			signal max_pulse	: out std_logic;
			signal dout 		: out std_logic_vector(3 downto 0)		
		);
	end entity bin_counter_4b;
	
	architecture rtl of bin_counter_4b is
		use ieee.numeric_std.all;
		constant NMAX	: unsigned(3 downto 0) := (others => '1');
		signal state_reg, state_next 	: unsigned(3 downto 0);
	begin
		--memory elements (registers)
		REG: process(clk, rst)
		begin
			if rst = '1' then 
				state_reg <= "0000"; 	
				--state_reg <= to_unsigned(0, state_reg'length);
				--state_reg	<= (others => '0');
			elsif rising_edge(clk) then 
				state_reg <= state_next;
			end if;
		end process REG;
		--next state logic
		NSL : state_next <= 
			"0000" 			when clr 	= '1' else
			unsigned (din)	when load 	= '1' else
			state_reg + 1 	when en 	= '1' else
			state_reg;
		--output logic
		OL1	: max_pulse 	<= 	'1' when state_reg = NMAX else 
								'0';
		OL2 : dour 			<= std_logic_vector(state_reg);
	end architecture rtl;




--Subprograms
--Subprograms > Procedures
--	Encapsulates a set of sequential statements that are executed when the procedure is called. 
--	Sequential or concurrent
--	May have in, out or inout arguments
process(...)
type real_vector is array (natural range <>) of real;
variable samples : real_vector(1 to 16);
	
procedure report_max_and_sum is 	--no arguments
begin
for i in samples'range loop
sum := sum + samples(i);
if samples(i) > max then
max := samples(i);
end if;
end loop;
report "Maximum value is: " & real'image(max);
report "Total value is: " & real'image(sum);
end procedure report_max_and_sum;
begin
--
report_max_and_sum;	--procedure call
end process;
-- 	This procedure executed each time the enclosing process is executed.

--Subprograms > Functions
--	Encapsulates a set of sequential statements that compute a result. 
--	Only sequential context.	
-- 	May only have mode in arguments.
--	Predefined functions: arithmetic operators (+,-, etc.), logical operators (and, or etc.)
architecture A of E is

--converts a vector to its integer value

```vhdl
function bv_to_nat(bv : in bit_vector) return natural is
variable result : natural := 0;
begin
for i in bv'range loop 
result := result*2 + bit'pos(bv(i));
end loop;
return result;
end function bv_to_nat;
 begin
data_nat1 	<= bv_to_nat(data_bv1);
data_nat2 	<= bv_to_nat(bv => data_bv2);	
 end architecture A;
```

## Package

### Package declaration:

Declarations of constants, types, signals, subprograms, files, components

### Package body:

Definition of deferred constants 

Subprogram bodies:

```vhdl
package example_pkg is 
constant MAX 		: integer := 10;
constant MAX_SIZE 	: natural;
subtype bv10 is bit_vector(MAX-1 downto 0);
procedure proc(A : in bv10; 	B : out bv10);
function func(A, B : in bv10) return bv10;
end package example_pkg;
```

```vhdl
package body example_pkg is
constant MAX_SIZE : natural := 200;
	
procedure proc(A : in bv10; 	B : out bv10) is
begin
B := abs(A);
end procedure proc;
	
function func(A, B : in bv10) return bv10 is
variable V : bv10;
begin
V := A and B;
return (not(V));
end function func;
end package body example_pkg;
```

```vhdl
use work.example_pkg.all;	--assuming that the package units have been analyzed in the logical library 'work'
```

Predefined attiributes:

```vhdl
A'LEFT       --is the leftmost subscript of array A or constrained array type.
A'RIGHT(N)   --is the rightmost subscript of dimension N of array A.
A'RANGE      --is the range  A'LEFT to A'RIGHT  or  A'LEFT downto A'RIGHT .
T'IMAGE(X)   --is a string representation of X that is of type T.
T'VALUE(X)   --is a value of type T converted from the string X.
T'POS(X)     --is the integer position of X in the discrete type T.
```

## RTL methodology

In RTL design, a circuit's behavior is defined in terms of the flow of signals (or transfer of data) between synchronous registers, and the logical operations performed on those signals.

From algorithm to hardware : From pseudo-code to algorithmic model

Sequential flow of execution: order of statements is important

Use of variables

May not be (efficiently) synthesizable. Variable may be optimized out, mapped to signals, or mapped to registers.

From algorithm to hardware :: From pseudo-code to dataflow model

* Concurrent behaviour
* Purely combinational logic

** Only possible for simple algorithms
** Not easily scalable on the size of arrays
** Support of control (loops, conditions) is limited
** Need to also support synchronous behaviour

<a name="sync"/>

## Synchronous design


* Glitches do not compromise functionality provided that the combinational logic outputs are stable at least during a T_setup time before the next active clock edge.



* Clock and (async) reset are the only signals that must be glitch-free.



* Std cell providers guarantee that register's hold times are smaller than their clock-to-q delays.


## Register transfer methodology

* Block diagram of the hardware is the foundation of every RTL design (need to think in boxes)
* RT methodology provides a formal framework for efficiently mapping algorithms to hardware.
* Key elements



* **Registers**	: store data and represent variables of an algorithm
* **Control unit**	: controls register operations
* **Datapath unit** 	: realizes register operations
* * The `<=` notation means that the assignment is done at the next clock cycle
* * _next signals	: FF data inputs
* * _reg  signals 	: FF data outputs


### Example of RT operations:


* `R <- 1`
* `R <- R<<3 + 1`


### Generic RTL architecture 


* I/O ports

  * Data ports 	: receive/send data from/to environment
  * Control ports 	: receive/send status from/to environment


* Control unit


  * Controls datapath and manages the sequence of operations (potentially based on feedback from the datapath), but does not manipulate data itself
  * Selects operands, operations, and destination of results through control signals.
  * Takes conditions from external environment (e.g., start signal) or status signals from datapath.
  * Returns status to external environment (e.g., ready/done signal).
<a name="fom"/>

## Figures of Merit


* **Cycles per data item Γ :** number of computation cycles between releasing two subsequent data items.
* **Longest path delay `t_crit`:** the lapse of time required for data to propagate along the longest path. A circuit cannot function correctly unless `t_clk>t_crit`
* **Time per data item `T`:** the lapse of time between releasing two subsequent data items, e.g. in s/sample, ms/frame, or s/computation. `T=t_clk.Γ > t_crit.Γ`
* Data throughput Θ: `=1𝑇=𝑓𝑐1/T𝑙= f_clk/Γ` expressed in pixel/s, sample/s, frame/s, record/s, FFT/s, or the like.
* **Latency `L`:**, number of computation cycles from a data item entering a circuit until the pertaining result becomes available.
* **Circuit size `A`:** expressed in mm2 or GE (gate equivalents)
* Size-time product `AT`:, the hardware resources spent to obtain a given throughput. 𝐴𝑇AT =𝐴A/Θ
* **Energy per data item `E`** , the amount of energy dissipated for a given computation on a data item e.g. in pJ/MAC, nJ/sample, J/datablock, or in mWs/frame.
* **Energy-time product `ET`:** indicates how much energy gets spent for achieving a given throughput, e.g. in J/datablocks/s or mWs2/videoframe.



* RTL design tradeoffs

  * Resource allocation and scheduling (the fundamental tradeoff)
  * Each resource (combinational logic) can perform one operation per cycle
  * Resources can (only) be re-used from cycle to cycle

* Parallel (independent) operations can be:

  * Carried out in parallel (same cycle) with parallel resources, but this requires more area
  * Carried out one after the other on the same resource in subsequent cycles, but this requires more time (more clock cycles)


* Subsequent (dependent) operations can be carried out

  * in the same cycle by directly connecting multiple resources, but this increases the cycle time and requires more area
  * in subsequent cycles on the same resource, but this increases the number of cycles required

<a name="sta"/>

## Static Timing Analysis

Questions answered by static timing analysis


* How fast can we clock a circuit? (maximum clock frequency)



* What is the time window (relative to the clock) in which I can safely apply signals to the input? (stimuli application window)


sample the output of a circuit? (response acquisition window)


* Timing arc from input to output (between data signals)


Propagation delay

Contamination delay

Entire circuit (multiple paths)

Critical path: max (t_pd^logic)

Timing arc from clock to output (between clock and data)


* Propagation delay



* Contamination delay	


STA is based on timing requirements of synchronous elements.

Timing constraints relative to the clock:


* Data must be stable around the active clock edge (data call window)



* Setup time



* Hold time


<a name="snip"/>

Snippets
--------

### Counter

* Definitions:

	constant MY_CNT_LIM : C_IN1_WIDTH/C_OUT_WIDTH;
	signal my_cnt 		: unsigned(5 downto 0);
	signal my_cnt_en	: std_logic;



	circle_cnt_en	<= '1' when (busy = '1') and (mean_ack = '1') else
					   '0';
	
	CIRCLE_CNT_PROC: process (iClk) 		-- 0...5
	begin
		if rising_edge(iClk) then
			if iRst = '1' or floor_cnt_en = '1' then
				circle_cnt 	<= (others => '0');
			elsif circle_cnt_en = '1' then
				circle_cnt 	<= circle_cnt + 1;
			end if;
		end if;
	end process CIRCLE_CNT_PROC;






