Inside FCL.sol, in function ecdsa_verify line 51
where checking R >= N to validate if R will be lesser
to N due to subtraction process in line 67
is semantically incorrect
because subtracting equalled R and N will only
result to 0 which is acceptable uint value.

Recommendation
Remove the = check in R and N
