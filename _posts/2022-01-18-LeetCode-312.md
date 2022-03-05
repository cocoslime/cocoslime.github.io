---
title:  "leetcode Problem - 312. Burst Balloons 풀이"
excerpt: "leetcode Problem 312"

categories:
  - Blog
tags:
  - Algorithm
  - leetcode
last_modified_at: 2021-01-18T21:11:00-05:00

toc: true
toc_sticky: true
---

leetcode 사이트의 312번 Burst Balloons 문제에 대한 풀이 및 해결법에 접근하는 단계에 대한 설명을 공유하고자 한다.

# leetcode - 312. Burst Balloons 풀이

문제 링크 : [https://leetcode.com/problems/burst-balloons/](https://leetcode.com/problems/burst-balloons/)

본 포스팅은 영어 원문 글을 바탕으로 번역하여 작성하였고 원본은 다음과 같다. 

- 원본 글 출처 : [https://leetcode.com/problems/burst-balloons/discuss/1659268/C%2B%2B-EASY-TO-UNDERSTAND-oror-All-Intuitions-step-by-step-with-detailed-explanations](https://leetcode.com/problems/burst-balloons/discuss/1659268/C%2B%2B-EASY-TO-UNDERSTAND-oror-All-Intuitions-step-by-step-with-detailed-explanations)

## 문제 재 정의

N개의 풍선이 주어져 있고, 각각의 풍선에는 점수가 할당되어 있으며 이는 nums 배열의 값과 같다. 풍선을 하나 터뜨렸을 때, 얻게되는 점수는 양 옆의 풍선의 값과 터진 풍선의 값을 곱한 값이다. 

```jsx
예를 들어, nums = [2,3,4,5] 에서
점수가 3 인 두 번째 풍선을 터뜨린다면
얻게되는 점수는 2 * 3 * 4 = 24 점 이다.
```

결국 모든 풍선을 터뜨렸을때, 얻을 수 있는 최대의 점수는 몇점인지 구하는 것이 문제이다.

- Edge case : 오른쪽이나 왼쪽의 이웃 풍선이 존재하지 않는 경계에 있는 풍선을 터뜨릴 경우는, 경계 바깥쪽에 가상의 1점 풍선이 있다고 가정하고 계산한다.

## 첫번째 단계

먼저 생각할 수 있는 방법은 백트래킹이다. 하지만, 백트래킹 방식으로 풀이하면, 복잡도는 O(n!)이 된다. n은 최대 500의 값을 가지므로 다른 방법을 생각해 보아야 한다.

## 두번째 단계

백트래킹 방식으로 풀어나가기 힘드므로, 자연스럽게 다이나믹 프로그래밍(DP)를 생각해 볼 수 있다. 그럼 이제는, 이 문제에서 DP를 어떻게 구현해야하는 가에 대한 고민이 필요하다. 

하지만 DP에서도 매력적인 접근법이 당장 머리에서 떠오르지 않지만, 다음과 같은 사실을 이용해 볼 수 있을 것 같다.

```cpp
 이미 터뜨려진 풍선은 앞으로 미래에 터뜨릴 풍선들의 최대 점수를 구하는 계산에서 무시한다.
```

## 세번째 단계

(이해하기 힘들었던 단락이라 원문을 남겨둡니다.)

Now to think for time complexity, we have C(n,k) cases for k balloons and for each case it need to scan the k balloons to compare. Still it's too high but It's better than O(n!) but worse than O(2^n) or maybe O(3^n).

## 네번째 단계

이 문제는 많은 유사한 서브 문제들로 이루어진 것 처럼 보인다. 서브 문제를 정의하고 해결하는 방법을 고민하며 시간 복잡도를 줄여가면 해결법을 찾을 수 있을 듯 하다.

## 다섯번째 단계

**i ..........k-1....k...k+1.......... j**

**[- - - - - - - - - - - - - - - - - -]**

우리는 K번째 풍선을 터뜨려 (k-1) 번째 풍선과 (k+1) 번째 풍선을 곱하여 점수를 얻었다고 가정하자. 이제 K번째 풍선은 터져서 없어졌으므로, 우리는 (i, k-1), (k+1, j) 의 두 개의 서브 문제를 해결하면 되지 않을까? 그러나, 이것은 잘못된 접근법이다. (i, k-1)과 (k+1, j)의 서브 문제는 k번째 풍선이 터진 이후에는 독립적이지 않은데, 왜냐하면 k-1번째 풍선과 k+1번째 풍선이 서로 인접하게 되고, 최대 점수를 계산하기 위해 서로가 필요할 것이다.

그래서 우리는 이 문제의 새로운 이슈에 대해 직면했다. 새로운 이슈는 터진 풍선의 왼쪽과 오른쪽 풍선이 인접하게 되고 이는 미래의 계산에 영향을 줄 것이다. 우리는 이러한 상황에서 해결하기 보다는, 이러한 상황이 만들어지지 않았다고 가정하고 문제를 해결해야만 한다.

## 여섯번째 단계

단 하나의 경우를 제외하고, 위의 이슈는 모든 k번째 풍선에서 일어날 것이다.  그 단 하나의 예외는 예를 들어 우리가 k번째 풍선을 마지막으로 터뜨린다면, k-1번째 풍선과 k+1번째 풍선은 최대 점수 계산을 위해서 서로를 필요로 하지 않으므로 (이미 터져있으므로) 이 서브 문제들은 독립적인 문제가 된다.

즉, 먼저 터뜨려지는 풍선을 생각해 시간 순서대로 문제를 풀이하지 말고, 시간에 역행하여 주어진 풍선들에서 마지막으로 터질 풍선을 먼저 골라간다면, 독립적인 서브 문제들을 얻을 수 있다.

## 일곱번째 단계

우리가 인접한 풍선을 확신할 수 있는 경우는 k번째 풍선이 첫번째로 선택되거나, 마지막으로 터뜨려질때이다. 첫번째라면 (nums[k-1] * nums[k] * nums[k+1])의 점수를, 마지막이라면 (nums[-1] * nums[k] * nums[n])의 점수를 얻게 될 것이다.

N개의 풍선이 있다고 가정하고 그 중 마지막에 터질 풍선을 j라고 하자. 우리는 j를 기준으로 두 개의 부분으로 풍선들을 나눌 수 있으며, j는 마지막에 터질 풍선이므로 두 개의 서로 독립적인 서브 문제로 나눌 수 있다.

## 정리

- **Edge case:** 배열의 양 끝에 가상의 1의 값을 추가한다. 이것은 최종 결과에 전혀 영향을 끼치지 않으며, 양 끝에서의 특수 케이스를 고려하지 않도록 해준다.
- 문제에서 일어나는 과정을 반대로 생각하는 것(마지막에 터지는 풍선 을 먼저 생각한다.)은 깔끔하게 배열을 서브 문제로 나눌 수 있도록 해준다.
- 0의 값을 가진 풍선들을 먼저 배열에서 제거 한 후 진행해도 된다.
- 재귀 함수 내에서 터지는 풍선의 점수를 계산할 때 사용할 양 옆 풍선은 배열의 양 끝으로 고정한다. **(배열의 양 끝 풍선들은 터지는 풍선으로 선택되지 않는다.)** 이 터지는 풍선이 이 배열내에서 마지막으로 터진다면, 그 때의 풍선의 양 옆은 배열의 양 끝이 될 것이기 때문이다.
- 각각의 선택된 풍선들은 다음 재귀 함수의 왼쪽 끝 혹은 오른쪽 끝이 된다. 이런 방식으로 재귀함수를 진행하고 기저 조건은 왼쪽 끝과 오른쪽 끝 사이에 풍선이 하나도 없을 경우이다. (left + 1 == right)

## 코드

원문의 코드는 Bottom-Up 방식으로 해결한 코드이다. 필자는 조금 더 이해하기 편한 Top-Down 방식으로 구현하여 아래 코드를 첨부하였다. 

```cpp
class Solution{
public:
	vector<vector<int>> cache;
	vector<int> new_nums;
	
	int dp(int start, int end){
	    if (start + 1 == end) return 0;
	
	    int& value = cache[start][end];

			// 이미 테이블에 값이 설정되어 있으면 바로 해당 값 리턴
	    if (value != -1)
	        return value;
	
	    value = 0;
	    for (int k = start + 1 ; k < end ; k++){
					// (start, end) 의 풍선 중에서 k 풍선을 마지막으로 선택한다.
	        value = max(value, dp(start, k) + dp(k, end) + new_nums[start] * new_nums[k] * new_nums[end]);
	    }
	    return value;
	}
	
	int maxCoins(vector<int>& nums) {
	    int n = nums.size() + 2;

			// DP 테이블을 초기화 한다.
	    cache.assign(n, vector<int>(n, -1));

			// 양쪽 경계에 가상의 풍선(1)을 추가한다.
	    new_nums.assign(n,1);
	    for (int i = 0 ; i < nums.size() ; i++){
	        new_nums[i+1] = nums[i];
	    }
	
	    return dp(0, n - 1);
	}
}
```