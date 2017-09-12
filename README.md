Copyright Notice

Copyright (c) 2016 Seagat2011 and the persons identified as the
document authors.  All rights reserved.

The following is a Request For Comment (RFC) for a proposed encoding scheme which encodes any 16K image or video frame using a method known as RE encapsulation. RE encapsulation is a realization of Chomsky's Finite Automata, and is a well-known method of string-matching, known to those skilled in the art. This RFC attempts to employ an RE encapsulation method to frame encoding. Code examples are demonstrated using pseudo code or ECMA-262 JAVASCRIPT

The proposed method is explained herein:

RULE  
1. A byte or bit is termed a pattern  
2. All tables reside in Read/Write memory  
3. Saved values for CLOCK must be accessible in both binary and integer form  
4. All provided decoding methods produce equivalent results having a trade-off in performance and memory space  
  
    PROCEDURE (16K ENCODING)  
    1. copy every byte or bit into a table,P[]  
    2. create an N-bit clock,CLOCK,having a sufficient resolution to reach every pattern,P[CLOCK],in the table P[]  
    3. create a temporary hash table,16K_STRING[],to be used as a condensing library  
    4. starting from CLOCK equal 0 at the beginning of table,P[],until end-of-table:
  
      check if the pattern,P[CLOCK],already exists in the table,16K_STRING[],and  
  
        If the pattern,P[CLOCK],does not exist in the table,16K_STRING[]:  
          add the pattern,P[CLOCK],to the table,and create another empty hash sub table inside it: 16K_STRING_SUBPATTERN[]   
          (eg. 16K_STRING[ P[CLOCK] ] = new 16K_STRING_SUBPATTERN[]) ; or  
        If the pattern,P[CLOCK],does indeed exist in the table,16K_STRING[],  
          save the CLOCK & the pattern,P[CLOCK],in the hash sub table 16K_STRING_SUBPATTERN[]. (eg 16K_STRING[ P[CLOCK] ].push(CLOCK) )   
  
      loop to next pattern in P[] (ie. add 1 to CLOCK) 
  
    5. stop at end-of-file (table)  
    6. save final resolution (#bits required for the clock) into a variable,RESOLUTION  
   
At this point we have built a pattern table where each clock value will generate a specific pattern.  
Now lets examine one of the sub patterns in 16K_STRING_SUBPATTERN[]  
Let say, for example, a 1-bit pattern was generated at the following clock values in the file  
  
1: 100|1100|11100|111100|1111100|11111100  
   
A lookup table will return the equivalent Regular Expression: /^1+00$/  
 
All RE (Regular Expression) patterns are distilled in a hash lookup table: 16K_RegExp_string[]  

    RE alphabet (3-bit) - 0,1,+,\d,(,),\1,|
  
We have now seen how to encode an N-bit clock  
  
  
    PROCEDURE (16K DECODING)  
    1. extract one RE pattern,RE,from a saved table,P[]  
    2. extract one pattern,PATTERN,from RE[]
    3. create an N-bit clock,CLOCK,specified by the variable,RESOLUTION,having a sufficient resolution to generate every required pattern,P[ RE[CLOCK] ],in the table P[ RE[] ]  
    4. create another temporary hash table,16K_STRING[],to be used as an image buffer  
    5. starting from CLOCK equal 0 at the beginning of table,P[ RE[] ],until end-of-table:  
      
      check if the pattern,CLOCK,matches,P[ RE[CLOCK] ],in binary number form,and  
   
        If the pattern,P[ RE[CLOCK] ],does not match CLOCK, then loop to next CLOCK (ie. add 1 to CLOCK); or  
        If the pattern,P[ RE[CLOCK] ],does indeed match CLOCK, then  
          write the value of PATTERN & CLOCK into the image buffer 16K_STRING. (eg 16K_STRING[ CLOCK ] = PATTERN )
   
      repeat above procedures until end-of-CLOCK
      reset CLOCK
  
    6. loop to next RE pattern in P[]  
    7. send image 16K_STRING to frame buffer  
    

RE patterns may be decoded in sequence or in parallel. On average, compression ratios per bit are greater than 1000:1

A further optimization can be made to decode performance by observing 16K_STRING[ P[CLOCK] ] has all the required bit patterns, and thus there really is no need to regenerate the clock,if,space permitting, a reverse-lookup hash table,16K_STRING_SUBPATTERN_rev[], is provided during decoding. 

  
    PROCEDURE (16K DECODING) (version 2) 
    1. extract one RE pattern,RE,from a saved table,P[] 
    2. extract one pattern,PATTERN,from RE[]
    3. create a temporary hash table,16K_STRING[],to be used as an image buffer   
    4. extract the corresponding CLOCKS from a provided REVERSE hash lookup table,16K_RegExp_string_rev[RE[PATTERN]]
    5. write the value of PATTERN, at all CLOCKS[ i=0 to N ], into the image buffer 16K_STRING. (eg 16K_STRING[ CLOCK[i] ] = PATTERN ) 
    6. loop to next pattern in P[] until end-of-table  
    7. send image 16K_STRING to frame buffer   
     
  
RE patterns may be decoded in sequence or in parallel. On average, compression ratios per bit are greater than 1000:1
