---
layout: page
name: AI 프로그래밍
tools: [C++, A* Algorithm]
image: "/assets/images/background/ai.JPG"
---

# AI Programming

<br>
{% include elements/video.html id="LgG7_HTG_Jk" %}

<br>

### A* 알고리즘

###### 교수님이 제공한 프레임워크에서 A* 알고리즘을 사용하여 패스 파인딩을 구현하였습니다.

###### 길을 찾는 업데이트 함수에서, while 반복문을 통해 openList에 저장된 노드들이 없어질 때까지 반복시킵니다. 현재 노드가 목표 노드와 일치하다면 COMPLETE를 리턴시키고, 아직 시스템이 처리 중이라면 PROCESSING, 만약 모든 노드들을 탐색했지만 목표 노드에 도달하지 못하였다면 IMPOSSIBLE을 리턴시키도록 하였습니다.

```c++
while (!openList.empty())
{
    parent_node = openList[0];
    openList.erase(openList.begin());
    parent_pos = GetGridPosFromPosition(parent_node.pos);
    
    if (goal_pos == parent_node.pos)
    {
        //node is the goal node.
        closeList.push_back(parent_node);
        UpdatePathFromCloseList(request);

        if (request.settings.rubberBanding && request.settings.smoothing)
        {
            RubberBanding(request);
            UpdatePath(request);
            AddPointsBack(request);
            Smoothing(request);
        }
        else if (request.settings.rubberBanding)
        {
            RubberBanding(request);
            UpdatePath(request);
        }
        else if (request.settings.smoothing)
        {
            UpdatePath(request);
            Smoothing(request);
        }
        else
        {
            UpdatePath(request);
        }

        if (request.settings.debugColoring)
        {
            terrain->set_color(parent_pos, Colors::Yellow);
        }

        return PathResult::COMPLETE;
    }

    for (int i = 0; i < 8; ++i)
    {
        NeighborChildNodes(i, parent_pos, parent_node.given, request.settings.heuristic, request.settings.weight, request.settings.debugColoring);
    }
    closeList.push_back(parent_node);
    listType_InBoard[parent_node.pos] = CLOSELIST;

    if (request.settings.debugColoring)
    {
        terrain->set_color(parent_pos, Colors::Yellow);
    }

    if (request.settings.singleStep)
    {
        return PathResult::PROCESSING;
    }

}

if (openList.empty())
{
    return PathResult::IMPOSSIBLE;
}

```

###### NeighborChildNodes 함수에서는 주변의 다른 노드들의 위치를 확인하고 각각의 비용에 대해서 계산을 했습니다.

```c++
    int curr_x = curr.col;
    int curr_y = curr.row;
    int parent_pos = curr_x + curr_y * width;
    int pos;
    float new_given_cost = given_cost;

    //find curr position and update given cost
    if (way == 0) // N
    {
        ++curr_y;
        ++new_given_cost;
    }
    else if (way == 1) // NE
    {
        ++curr_x;
        ++curr_y;
        new_given_cost += DIAGONAL_COST;

        //check the curr grid is corner, check that grid is valid and then check left & bottom
        if (terrain->is_valid_grid_position({ curr_y, curr_x - 1 }) && terrain->is_valid_grid_position({ curr_y - 1, curr_x }))
        {
            if (terrain->is_wall({ curr_y, curr_x - 1 }) || terrain->is_wall({ curr_y - 1, curr_x }))
            {
                return;
            }
        }
    }
    else if (way == 2) // E
    {
        ++curr_x;
        ++new_given_cost;
    }
    else if (way == 3) // SE
    {
        ++curr_x;
        --curr_y;
        new_given_cost += DIAGONAL_COST;

        //check the curr grid is corner, check that grid is valid and then check left & top of that grid
        if (terrain->is_valid_grid_position({ curr_y, curr_x - 1 }) && terrain->is_valid_grid_position({ curr_y + 1, curr_x }))
        {
            if (terrain->is_wall({ curr_y, curr_x - 1 }) || terrain->is_wall({ curr_y + 1, curr_x }))
            {
                return;
            }
        }
    }
    else if (way == 4) // S
    {
        --curr_y;
        ++new_given_cost;
    }
    else if (way == 5) // SW
    {
        --curr_x;
        --curr_y;
        new_given_cost += DIAGONAL_COST;

        //check the curr grid is corner, check that grid is valid and then check right & top
        if (terrain->is_valid_grid_position({ curr_y, curr_x + 1 }) && terrain->is_valid_grid_position({ curr_y + 1, curr_x }))
        {
            if (terrain->is_wall({ curr_y, curr_x + 1 }) || terrain->is_wall({ curr_y + 1, curr_x }))
            {
                return;
            }
        }
    }
    else if (way == 6) // W
    {
        --curr_x;
        ++new_given_cost;
    }
    else if (way == 7) // NW
    {
        --curr_x;
        ++curr_y;
        new_given_cost += DIAGONAL_COST;

        //check the curr grid is corner, check that grid is valid and then check right & bottom
        if (terrain->is_valid_grid_position({ curr_y, curr_x + 1 }) && terrain->is_valid_grid_position({ curr_y - 1, curr_x }))
        {
            if (terrain->is_wall({ curr_y, curr_x + 1 }) || terrain->is_wall({ curr_y - 1, curr_x }))
            {
                return;
            }
        }
    }
```

###### 그런 후 새로운 비용과 기존 비용을 합하여 업데이트 시킵니다. 만약 openList라는 찾아야 할 노드들에 이 노드가 존재할 경우 가장 비용이 싼 노드를 추가시키고 비싼 노드는 삭제합니다. 만약 closeList라는 이미 찾았던 노드들에 이 노드가 존재할 경우 비용을 확인하고 싸다면 openList에 넣도록 하였습니다.
``` c++
    int openList_size = openList.size();
    float cost = new_given_cost + CalculateHeuristic({ curr_y, curr_x }, goal, heuristic) * weight;

    if (listType_InBoard[pos] == NONELIST)
    {
        if (debuging_color)
        {
            terrain->set_color({ curr_y, curr_x }, Colors::Blue);
        }

        InsertNodeOnOpenList(NodeInitialize(parent_pos, pos, cost, new_given_cost));

        listType_InBoard[pos] = OPENLIST;
    }
    else if (listType_InBoard[pos] == OPENLIST) //the grid is on open list 
    {
        //find the grid on open list and then compare the old cost and new cost
        for (std::vector<Node>::iterator it = openList.begin(); it != openList.end(); ++it)
        {
            //find the grid on open list
            if ((*it).pos == pos)
            {
                //if the new cost is cheaper than the old cost, then delete old one
                if ((*it).cost > cost)
                {
                    openList.erase(it);

                    //add new cheaper one
                    InsertNodeOnOpenList(NodeInitialize(parent_pos, pos, cost, new_given_cost));
                }
                break;
            }
        }
    }
    else //check the grid is on close list
    {
        //find the grid on close list
        for (std::vector<Node>::iterator it = closeList.begin(); it != closeList.end(); ++it)
        {
            if ((*it).pos == pos)
            {
                //compare the old cost and enw cost, if the new cost is cheaper, delete old one
                if ((*it).cost > cost)
                {
                    closeList.erase(it);
                    //put new one on open list

                    InsertNodeOnOpenList(NodeInitialize(parent_pos, pos, cost, new_given_cost));
                }
                break;
            }
        }
    }
```

###### 프로젝트 내에서 루트를 부드럽게 움직이도록 지시하였습니다. 저는 Catmull Rom 스플라인을 사용하여 곡선을 구현하였습니다. 구현 식은 [곡선&스플라인](https://tsyang.tistory.com/57)에서 참고를 하였습니다.
```c++
Vec3 AStarPather::Catmull_Rom_Spline(Vec3 point_1, Vec3 point_2, Vec3 point_3, Vec3 point_4, float s)
{
    Vec3 output_point;
    float three_times = std::pow(s, 3);
    float two_times = std::pow(s, 2);

    output_point = point_1 * (-0.5 * three_times + two_times - 0.5 * s) + point_2 * (1.5 * three_times - 2.5 * two_times + 1.0) +
        point_3 * (-1.5 * three_times + 2.0 * two_times + 0.5 * s) + point_4 * (0.5 * three_times - 0.5 * two_times);

    return output_point;
}
```

###### 이 프로젝트를 통해서 전반적인 A* 알고리즘에 대해서 공부할 수 있었습니다.