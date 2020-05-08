---
layout: post
title: HackerRank. Insert a node at a specific position in a linked list
tags: Algorithm HackerRank LinkedList Java
categories: Algorithm
---
  
* TOC
{:toc}  
  
## Problem
[https://www.hackerrank.com/challenges/insert-a-node-at-a-specific-position-in-a-linked-list/problem](https://www.hackerrank.com/challenges/insert-a-node-at-a-specific-position-in-a-linked-list/problem)    
  
<br>  

## Think
* LinkedList의 삽입을 직접 구현하는 문제이다.
* ArrayList와 같이 Index 기반이 아니라
* Next Node의 정보만 가지고 있다는
* LinkedList의 특성을 알고있으면 쉽게 풀 수 있다.

<br>  

## Solved Code
  
```java  
// Complete the insertNodeAtPosition function below.
   
    static SinglyLinkedListNode insertNodeAtPosition(SinglyLinkedListNode head, int data, int position) {

        int count = 0;
        SinglyLinkedListNode prevNode = null;
        SinglyLinkedListNode curNode = head;
        
        while(curNode != null) {
            if(count == position) {
                SinglyLinkedListNode singlyLinkedListNode = new SinglyLinkedListNode(data);
                
                if(prevNode != null) {
                    prevNode.next = singlyLinkedListNode;
                }
                singlyLinkedListNode.next = curNode;

                break;
            } else {
                prevNode = curNode;
                curNode = curNode.next;
                count++;
            }
        }

        return head;
    }
```  
