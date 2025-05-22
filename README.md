# Hawk High - Findings Report

# Table of contents
- ## [Contest Summary](#contest-summary)
- ## [Results Summary](#results-summary)
- ## High Risk Findings
    - ### [H-01. Broken Invariant in `graduateAndUpgrade` function . Missing check for the `cutoff` criteria . ](#H-01)
    - ### [H-02. Incorrect Mathmatical calculation in function `graduateAndUpgrade` . ](#H-02)

- ## Low Risk Findings
    - ### [L-01. Missing check for `principal` in function `addTeacher` ](#L-01)


# <a id='contest-summary'></a>Contest Summary

### Sponsor: First Flight #39

### Dates: May 1st, 2025 - May 8th, 2025

[See more contest details here](https://codehawks.cyfrin.io/c/2025-05-hawk-high)

# <a id='results-summary'></a>Results Summary

### Number of findings:
- High: 2
- Medium: 0
- Low: 1


# High Risk Findings

## <a id='H-01'></a>H-01. Broken Invariant in `graduateAndUpgrade` function . Missing check for the `cutoff` criteria .             



## Summary

As per the docs , any student who not have achived the `cutoff` criteria will not be upgraded . By missing the check for `cutoff` criteria we allow student to upgrade without event meeting the `cutoff` criteria .&#x20;

## Vulnerability Details

As per the docs , any student who not have achived the `cutoff` criteria will not be upgraded . By missing the check for `cutoff` criteria we allow student to upgrade without event meeting the `cutoff` criteria .

<https://github.com/CodeHawks-Contests/2025-05-hawk-high/blob/3a7251910c31739505a8699c7a0fc1b7de2c30b5/src/LevelOne.sol#L295>

```Solidity
    function graduateAndUpgrade(address _levelTwo, bytes memory) public onlyPrincipal {
        if (_levelTwo == address(0)) {
            revert HH__ZeroAddress();
        }

        uint256 totalTeachers = listOfTeachers.length;// we dosent check for the cutoffscore, if any student dont achived that, upgrade should not be happening & if anybody have given <5 review 

        uint256 payPerTeacher = (bursary * TEACHER_WAGE) / PRECISION; 
        uint256 principalPay = (bursary * PRINCIPAL_WAGE) / PRECISION;// q incorrect mathmatical calculation 

        _authorizeUpgrade(_levelTwo); // q cannot upgrade is sesson is not ended 

        for (uint256 n = 0; n < totalTeachers; n++) {
            usdc.safeTransfer(listOfTeachers[n], payPerTeacher); // q if one teacher gets blacklist no one will get the reward 
        }

        usdc.safeTransfer(principal, principalPay);
    }
```

## Impact

Allow student to upgrade without even achiveing the `cutoff` criteria .&#x20;

## Tools Used

Manual review&#x20;

## Recommendations

check for the \`cutoff\` criteria, if student not achived that , then revert . &#x20;

## <a id='H-02'></a>H-02. Incorrect Mathmatical calculation in function `graduateAndUpgrade` .             



## Summary

In function `graduateAndUpgrade` we are calculating `payperTeacher` & `principalPay` . But we are calculating incorrectly, `bursary` is the total fund of contract. if we multiply `bursary` by `TEACHER_WAGE` OR `PRINCIPAL_WAGE` . we will get the husge amount, which will be greater than whole bursary fund. Paying this much of wage can lead to loss of funds.

## Vulnerability Details

In function `graduateAndUpgrade` we are calculating `payperTeacher` & `principalPay` . But we are calculating incorrectly, `bursary` is the total fund of contract. if we multiply `bursary` by `TEACHER_WAGE` OR `PRINCIPAL_WAGE` . we will get the husge amount, which will be greater than whole bursary fund. Paying this much of wage can lead to loss of funds.

<https://github.com/CodeHawks-Contests/2025-05-hawk-high/blob/3a7251910c31739505a8699c7a0fc1b7de2c30b5/src/LevelOne.sol#L302>

<br />

<https://github.com/CodeHawks-Contests/2025-05-hawk-high/blob/3a7251910c31739505a8699c7a0fc1b7de2c30b5/src/LevelOne.sol#L303>

<br />

```Solidity
        uint256 payPerTeacher = (bursary * TEACHER_WAGE) / PRECISION; 
        uint256 principalPay = (bursary * PRINCIPAL_WAGE) / PRECISION;
```

```solidity
    function graduateAndUpgrade(address _levelTwo, bytes memory) public onlyPrincipal {
        if (_levelTwo == address(0)) {
            revert HH__ZeroAddress();
        }

        uint256 totalTeachers = listOfTeachers.length;// we dosent check for the cutoffscore, if any student dont achived that, upgrade should not be happening & if anybody have given <5 review 

        uint256 payPerTeacher = (bursary * TEACHER_WAGE) / PRECISION; 
        uint256 principalPay = (bursary * PRINCIPAL_WAGE) / PRECISION;// q incorrect mathmatical calculation 

        _authorizeUpgrade(_levelTwo); // q cannot upgrade is sesson is not ended 

        for (uint256 n = 0; n < totalTeachers; n++) {
            usdc.safeTransfer(listOfTeachers[n], payPerTeacher); // q if one teacher gets blacklist no one will get the reward 
        }

        usdc.safeTransfer(principal, principalPay);
    
```

## Impact
This much transfer of funds to the teacher or principal can lead to loss of funds . 

## Tools Used
Manual Review 

## Recommendations
Correct the formula . 



    


# Low Risk Findings

## <a id='L-01'></a>L-01. Missing check for `principal` in function `addTeacher`             



## Summary

In function `addTeacher` we are missing check that is `msg.sender = principal `  . Principal can add itself as a teacher and withdraw the salary of teacher and principal both .&#x20;

## Vulnerability Details

In function `addTeacher` we are missing check that is `msg.sender = principal ` . Principal can add itself as a teacher and withdraw the salary of teacher and principal both .

<https://github.com/CodeHawks-Contests/2025-05-hawk-high/blob/3a7251910c31739505a8699c7a0fc1b7de2c30b5/src/LevelOne.sol#L201>

```Solidity
    function addTeacher(address _teacher) public onlyPrincipal notYetInSession {
        if (_teacher == address(0)) {
            revert HH__ZeroAddress();
        }

        if (isTeacher[_teacher]) {
            revert HH__TeacherExists();
        }

        if (isStudent[_teacher]) {
            revert HH__NotAllowed(); // q check that principal can also not set himself as teacher and takes salary 
        }

        listOfTeachers.push(_teacher);
        isTeacher[_teacher] = true;

        emit TeacherAdded(_teacher);
    
    }


```



## Impact

Principal can add itself as a teacher and can withdraw the salary or can manipulate the review of the student . Drawing salary as teacher and principal both can lead to the loss of funds to the protocol . 

## Tools Used
manual review 

## Recommendations

Add check . So, that if principal tries  to add itself as a teacher it will revert .  

```solidity
    function addTeacher(address _teacher) public onlyPrincipal notYetInSession {
        if (_teacher == address(0)) {
            revert HH__ZeroAddress();
        }

        if (isTeacher[_teacher]) {
            revert HH__TeacherExists();
        }

        if (isStudent[_teacher]) {
            revert HH__NotAllowed(); 
        }

       if (isPrincipal[_teacher]){
              revert HH__NotAllowed();
}

        listOfTeachers.push(_teacher);
        isTeacher[_teacher] = true;

        emit TeacherAdded(_teacher);
    }
```




