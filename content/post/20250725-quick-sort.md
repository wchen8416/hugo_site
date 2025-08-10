---
title: "quick sort algorithm"
subtitle: " "
description: " "
date: 2025-07-25T05:34:19Z
author:      "Wayne"
image: ""
tags: ["algorithm"]
categories: ["Tech"]
---

# demos

## c

```c
#include <stdio.h>
#include <time.h>
#include <stdlib.h>
#include <unistd.h>

void swap(int *x, int *y)
{
    if (x == y)
    {
        return;
    }

    int temp = *x;
    *x = *y;
    *y = temp;
}

void quick_sort(int array[], unsigned int left, unsigned int right)
{
    if (left >= right || NULL == array)
    {
        return;
    }

    int pivot = array[left];
    unsigned int i = left, j = right;

    while (i < j)
    {
        while (array[j] >= pivot && j > i)
        {
            j--;
        }

        if (j > i)
        {
            swap(&array[i], &array[j]);
            i++;
        }

        while (array[i] <= pivot && i < j)
        {
            i++;
        }

        if (i < j)
        {
            swap(&array[i], &array[j]);
            j--;
        }
    }

    array[i] = pivot;

    if (i > left)
    {
        quick_sort(array, left, i - 1);
    }
    quick_sort(array, i + 1, right);
}

void array_print(int array[], unsigned int size)
{
    for (unsigned int i = 0; i < size; i++)
    {
        printf("%6d ", array[i]);
    }
    printf("\n");
}

int main(int argc, char *argv[])
{
    const unsigned int size = 10;
    int array[size];
    srand(time(NULL));

    unsigned int loop = 10;

    while (loop > 0)
    {
        for (unsigned int i = 0; i < size; i++)
        {
            array[i] = rand() % 1000;
        }

        printf("排序前: ");
        array_print(array, size);
        quick_sort(array, 0, size - 1);
        printf("排序后: ");
        array_print(array, size);
        printf("\n");
        sleep(1);
        loop--;
    }

    return 0;
}
```

## c++

## Python

## Go

## Node.js

# complex analysis
