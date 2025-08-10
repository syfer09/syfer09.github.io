---
layout: default 
title: Round 1042 (Div. 3) End of Contest Breakdown
date:   2025-08-10 14:00:00 +0000
categories: jekyll update
permalink: /round1024div3summary
---

# Round 1024 (Div. 3) End of Contest Breakdown

## General Thoughts
[Problem Statements](https://codeforces.com/contest/2131);

First contest in a while, in fact this is my first time doing one since division 3 was added. Division 3 is supposed to be an easier version of the contests for people with a lower rating. 

Being really out of practice, I was quite happy to solve both A and B, however C ended up escaping me during the solution. After the contest ended I was able to see which test case failed and fix my code accordingly to get a submission. 

Although I am happy with getting two solutions, I solved C really early on as well, and spent the rest of the time trying to implement my solution.

## Problem A
### [Statement Summary](https://codeforces.com/contest/2131/problem/A)
Given two arrays $$a$$ and $$b$$, we are allowed to continously repeat two steps:

1. Choose a random index $$i$$ such that $$a_i>b_i$$. Then, decrease $$ai$$ by 1. If there does not exist such $$i$$, ignore this step.

2. Choose a random index $$i$$ such that $$a_i\lt b_i$$. Then, increase $$a_i$$ by 1. If there does not exist such $$i$$, ignore this step. 

The iterations stop once we fail to execute step 1. Find the number of iterations that will happen. It can be proved that the number of iterations will be the same regardless of the random indices chosen.

### Solution
Since $$i$$ has to be the same, we can just check the elements pairwise to find the biggest $$a_i - b_i$$. After that we know that our number of iterations is $$a_i - b_i + 1$$. The extra 1 comes from the fact that even if step 1 and step 2 will fail to execute that is still an iteration, and the next one will fail since before iterating we check if step 1 was executed on the *previous* iteration.

### Code
```
// Date: August 10th, 2025
// Author: syfer
// Problem: round1042Adiv3

#include <bits/stdc++.h>

using namespace std;

void solve() {
    int n;
    vector<int> a,b;
    cin >> n;
    for(int i = 0; i < n; i++){
        int a1;
        cin >> a1;
        a.push_back(a1);
    }
    for(int i = 0; i < n; i++){
        int b1;
        cin >> b1;
        b.push_back(b1);
    }
    int maxdiff = 0;
    for(int i = 0; i < n; i++){
        if(a[i] - b[i] > 0)
            maxdiff += a[i] - b[i];
    }
    cout << maxdiff + 1<< endl;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);

    int t;
    cin >> t;
    while (t--) solve();
}

```

## Problem B
### [Statement Summary](https://codeforces.com/contest/2131/problem/B)
We call an array $$a$$ good if each $$a_i * a_{i+1}$$ is negative, and every sub-array of consecutive elements has a positive sum.

For each array size $$n$$, find the smallest possible array that satisfies these conditions. We say array $$a$$ is smaller than $$b$$ if $$a_i$$ $$a_i\leq b_i$$ in the first place where $$a$$ and $$b$$ differ.

### Solution
We try to build each array from the start. The smallest first element that we can put is $$-1$$. If we put $$1$$ as the first element, the second would have to be negative, and the subarray of the first two elements not have a positive sum since the next element would have to be *at least* negative 1, which would give a sum of 0 (which not positive).

Assuming our first element is $$-1$$ the next element could be $$2$$ however, we then realize that the smallest possible third element would have to be $$-1$$ again, meaning our sum over the first three elements is 0. This means we have to put 3 as the second element. After that $$-1$$ and we can continue alternating until we reach $$n$$ elements. 

I submitted the above, and got a wrong answer on one of the tests. This made me realize that if $$n$$ is even, that means we end on a 3, which can actually be reduced to a 2, since we no longer have an element to worry about at the end to have to sum with positively, and $$-1 + 2$$ is positive. Fixing that, my code submitted fine.

### Code

```
// Date: August 10th, 2025
// Author: syfer
// Problem: round1042Bdiv3

#include <bits/stdc++.h>

using namespace std;

void solve() {
    int n;
    cin >> n;
    if(n == 2){
        cout << "-1" << " 2" <<endl;
    }else{
       for(int i = 0;i < n - 1;i++){
           if(i%2 == 0){
               cout<< "-1 ";
           } else{
               cout<< "3 ";
           }
       }
       if(n % 2 == 0){
           cout<< "2" <<endl;
       } else 
           cout << "-1" <<endl;
    }
    
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);

    int t;
    cin >> t;
    while (t--) solve();
}
```

## Problem C
### [Statement Summary](https://codeforces.com/contest/2131/problem/C)
Given two arrays $$S$$ and $$T$$, along with an integer $$k$$, we are allowed to perform two possible operations on $$S$$ with the goal of making it equal to $$T$$. 

Our first operation is to take some element in $$S$$ and add $$k$$ to it.

Our second operation is to take some element $$x\in S$$ and replace it with $$\lvert x-k\rvert$$.

For every given test case, write "Yes" or "No" as to whether it is possible to use any number of these operations to set $$S$$ equal to $$T$$.

Equal in this case means that every element of $$S$$ appears the same number of times in $$T$$ (i.e. there exists some permutation of $$S$$ such that $$S$$ and $$T$$ are the same array). 

### Solution
The interesting thing to consider here is that each element $$x\in S$$ can become any number of values of $$x + c * k$$ for some constant $$c$$. This means that any member $$y\in T$$ can be created from $$x$$ so long as $$x\equiv y\pmod{k}$$. We also notice that we can reduce $$x$$ to $$x\pmod{k}$$ through the same operations, and then subtract $$k$$ as through our second operation. Therefore $$x$$ can also generate all $$(k-x)\pmod{k}) * c$$, or equivalently generate $$y\in T$$ if $$((k-x)\pmod{k})\equiv y\pmod{k}$$.

Therefore we can just reduce both $$S$$ and $$T$$ modulo $$k$$, and work with the elements as such. In particular each element $$r$$ of the reduced $$S$$ can kill an element of the reduced $$T$$. Killing means we remove that element from $$T$$. If there are no remaining elements $$r$$ in reduced $$T$$, we could also use $$r$$ to kill the element $$k-r$$. Only after neither $$r$$ nor $$k-r$$ exist in $$T$$ can we conclude that it is not possible to make $$S$$ into $$T$$. If we are able to kill every element in $$T$$ and are left with an empty array we can conclude that it is possible to turn $$S$$ into $$T$$.

### Code

```
// Date: August 10th, 2025
// Author: syfer
// Problem: round1042Cdiv3

#include <bits/stdc++.h>

using namespace std;

void solve() {
    long n, k;
    cin >> n >> k;
    vector <long> s;
    unordered_map<long,long> hash;
    for(long i = 0; i < n; i++){
        long s1;
        cin >> s1;
        s.push_back(s1%k);
    }
    for(long i = 0; i < n; i++){
        long t1;
        cin >> t1;
        if(hash.find(t1 % k) != hash.end()) hash[t1 % k]++;
        else
            hash[t1 % k] = 1;
    }
    for(long i = 0; i < n; i++){
        if(hash.find(s[i]) != hash.end() && hash[s[i]] > 0) {
            hash[s[i]]--;
        }
        else if ((hash.find(s[i]) == hash.end()) ||
                (hash.find(s[i]) != hash.end() && hash[s[i]] == 0)){
            if(hash.find(k-s[i]) == hash.end() || (hash[k - s[i]]) == 0){
                cout<<"NO"<<endl;
                return;
            }else
                hash[k - s[i]]--;
        }
    }
    cout << "YES" <<endl;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);

    int t;
    cin >> t;
    while (t--) solve();
}
```

### Further Thoughts on C
This was a very frustrating problem. I realized the modulo trick within the first 10 minutes of looking at the problem, and rest was implementation. At first I was considering going through $$T$$ for every element in $$S$$ and doing it manually. This would however result in a complexity of $$O(n^2)$$ which would have surely failed  the timed tests.

Next I decided to hash, and count every element of $$T$$ in that hash. This is the code that I was able to eventually get working after the contest end. 

The main difficulty I had turned out to be a missing `%` sign. This bug took me an hour to track down. Thinking about it afterwards perhaps I took the wrong approach here to my debugging.

I'm quite used to debugging contest problems where the test cases are hidden, through the old fashioned method of staring at my code until I see a mistake. After all, I feel like I'm not going to get any more information than the code I already have written. This time, I decided to be proactive about my testing. As soon as my first submission failed, I started writing a generator that would create random cases for me. In particular I wanted all cases in which permutations were possible, so that I could see if I printed a "No" for any of those cases. Here is the generator code in C:

```
// Filename: generator.c
// Purpose: generates test cases for round1042C
// Author: syfer
// Date: August 10th, 2025

#include <stdio.h>
#include <stdlib.h>
#include <math.h>

#define MAX_SIZE 100000
#define CASES 10

int main(int argc, char *argv[]){

    if(argc != 2) return 0;

    srand((unsigned int)argv[1]);
    
    long t1 = CASES;
    printf("%ld\n", t1);
    while(t1--){
        long n = (rand() % MAX_SIZE) + 2;
        long k = (rand() % MAX_SIZE) + 2;

        long t[n];
        long s[n];

        for(long i = 0; i < n; i++){
            t[i] = (rand() % MAX_SIZE);
            if(rand()%2)
                s[i] = t[i] + (rand()% MAX_SIZE) * k + 1;
            else
                s[i] = fabs(t[i] - k) + (rand()% MAX_SIZE) * k;
        }
        printf("%ld %ld\n", n, k);
        for(long i = 0; i < n;i++)
            printf("%ld ", s[i]);
        printf("\n");
        for(long i = 0; i < n;i++)
            printf("%ld ", t[i]);
        printf("\n");
    }
    return 0;
}
```
Eventually through testing, I was able to find the cases that I failed in, and with 10 minutes left in the contest I found the missing `%` sign.

That however was not my only bug. Turns out I was missing an additional check, to print "No" if I was trying to kill an element whose occurences had already been reduced to 0. I was only checking to see if I was trying to kill an element that did not exist, and printing no in that case.

Now that I think about it, my generator was testing "Yes" cases only, and I did not have time to test the "No"'s meaning that I was getting false positives. Most likely if I had more time, I could have tested all the random "No" cases, and came away with a fix for that bug. 

Regardless I found it as I got access to the test that I failed.

Although I'm happy I got the solution in the end, my algorithm is still quite slow, and the solution given by the organizers is much cleaner. On the one hand I should have written less buggy code, on the other hand if my solution was simpler (no need for a hash), then I probably would have had less bugs overall.

Despite the fact that writing the generator took a long time, and I could have potentially found the bug earlier without it, I'm very happy with the results I got by using it. I was able to isolate individual test cases and check my code when it went wrong, without relying on having to submit every time I wanted to get test cases that were not part of the initial few (there is after all a penalty for every incorrect submission). 
