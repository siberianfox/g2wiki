###Algorithm Summary
- Generate an imaginary cylinder surrounding the projection of the target block (b0, b0proj)
- The radius of the cylinder is the tolerance for annealing the new block (b1)
- Calculate if b1's endpoint falls inside or outside of the cylinder
- If b1 endpoint falls inside then combine b1 into b0. Do not change the projection of the original b0
- If b1 endpoint falls outside then queue b0 and make b1 the new b0
- If no new blocks arrive after the new-block-timeout queue b0 (typ 50 ms)
	
In the following example 3 new blocks are received. The first 2 are annealed, the 3rd falls outside the tolerance
![](images/BlockAnnealing-2016-01-29.png)

###Definitions
- b0	Head block. Block being tested for additions		
  - b0_target	Target endpoint of B0. Changes with each new annealed block		
  - b0_unit	Unit vector of B0. May change with each new annealed block		
  - b0_unit0	Initial unit vector of B0. Does not change as blocks are added so error cylinder does not drift		
- b1	New block - just received and not yet planned		
  - b1_target	Target endpoint of b1		
  - b1_unit	Unit vector of b1		
  - Tol_linear	Linear tolerance. Allowable deviation from path. The radius of the cylinder		
  - Tol_rotary	Block rotary tolerance. Allowable deviation from the rotational path		
  - Length	The length of the new block		
  - a	The side of the triangle formed by the b0 projection		
  - b	The side of the triangle formed by the b1 vector		
  - c	The side of the triangle formed by the right angle of b0 projection to b1_target		

###Algorithm
- Perform the following tests on b1. If any fail, exit and queue the current b0 and make b1 the new b0			
  - (a) Exit if b1 exceeds a length threshold			
  - (b) Exit if b0 and b1 target velocities are different (beyond a very small threshold)			
  - (c) Exit if b1 exceeds a maximum direction change			
    - Generate the b1_unit			
    - Take the dot product of b1_unit and b0_unit0 to find the cosine of b0 and b1 (save this value)			
    - Exit if the cosine exceeds threshold (too much direction change)			
				
  - (d) Exit if b1_target lies outside the tolerance cylinder			
    - Find the length of the altitude (c side) of the triangle formed by the b0 projection ('a' side) and the hypotenuse formed by b1 ('b' side)			
    - c = Length(b1) * sin(b1, b0)			which can be expressed as…
    - c = L * sqrt(1 - cos^2)			where cos was determined from the dot product in test (c)
				
  - Perform similar tests for rotary axes if any of ABC are in the move			
  - If all tests pass, then anneal b1 into b0:			
    - Update b0_target to b1_target			
    - Recalculate the b0 unit vector			
				
###Notes and Discussion
Annealing the block resets the endpoint of the annealed line - i.e. washes out the previous endpoint. This will change the unit vector of the b0 block. In order to keep the "error cylinder" from drifting the initial unit vector is preserved and used to set the mid-line of subsequent new blocks.
