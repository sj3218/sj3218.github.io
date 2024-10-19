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


<details>
<summary>Pathfinding.cpp 전체 코드</summary>
<div markdown="1" style="padding-left:20px;">

```c++
#include <pch.h>
#include "Projects/ProjectTwo.h"
#include "P2_Pathfinding.h"

#pragma region Extra Credit
bool ProjectTwo::implemented_floyd_warshall()
{
    return false;
}

bool ProjectTwo::implemented_goal_bounding()
{
    return false;
}

bool ProjectTwo::implemented_jps_plus()
{
    return false;
}
#pragma endregion

bool AStarPather::initialize()
{
    // handle any one-time setup requirements you have

    /*
        If you want to do any map-preprocessing, you'll need to listen
        for the map change message.  It'll look something like this:

        Callback cb = std::bind(&AStarPather::your_function_name, this);
        Messenger::listen_for_message(Messages::MAP_CHANGE, cb);

        There are other alternatives to using std::bind, so feel free to mix it up.
        Callback is just a typedef for std::function<void(void)>, so any std::invoke'able
        object that std::function can wrap will suffice.
    */

    return true; // return false if any errors actually occur, to stop engine initialization
}

void AStarPather::shutdown()
{
    /*
        Free any dynamically allocated memory or any other general house-
        keeping you need to do during shutdown.
    */
}

GridPos AStarPather::GetGridPosFromPosition(int pos)
{
    GridPos grid_pos;
    grid_pos.col = pos % width;
    grid_pos.row = pos / width;
    return grid_pos;
}

Node AStarPather::NodeInitialize(GridPos parent, GridPos curr, float cost_, float given_)
{
    Node new_node;
    new_node.pos = curr.col + curr.row * width;
    new_node.pos_parent = parent.col + parent.row * width;
    new_node.cost = cost_;
    new_node.given = given_;

    return new_node;
}

Node AStarPather::NodeInitialize(int parent, int curr, float cost_, float given_)
{
    Node new_node;
    new_node.pos_parent = parent;
    new_node.pos = curr;
    new_node.cost = cost_;
    new_node.given = given_;
    return new_node;
}

float AStarPather::CalculateHeuristic(GridPos curr, GridPos goal, Heuristic heuristic)
{
    float value_heuristic = 0.f;
    float x_diff = abs(goal.col - curr.col);
    float y_diff = abs(goal.row - curr.row);

    if (heuristic == Heuristic::EUCLIDEAN)
    {
        value_heuristic = std::sqrt(std::pow(x_diff, 2.f) + std::pow(y_diff, 2.f));
    }
    else if (heuristic == Heuristic::OCTILE)
    {
        float min = std::min(x_diff, y_diff);
        value_heuristic = min * DIAGONAL_COST + std::max(x_diff, y_diff) - min;
    }
    else if (heuristic == Heuristic::CHEBYSHEV)
    {
        value_heuristic = std::max(x_diff, y_diff);
    }
    else if (heuristic == Heuristic::MANHATTAN)
    {
        value_heuristic = x_diff + y_diff;
    }

    return value_heuristic;
}

void AStarPather::NeighborChildNodes(int way, GridPos curr, float given_cost, Heuristic heuristic, float weight, bool debuging_color)
{
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
    pos = curr_x + curr_y * width;

    //if the grid position is not valid or is it wall, then return
    if (!terrain->is_valid_grid_position({ curr_y, curr_x }) || terrain->is_wall({ curr_y, curr_x }))
    {
        return;
    }

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
}

void AStarPather::RubberBanding(PathRequest& request)
{
    int start_index = 0;
    int end_index = 2;

    GridPos start_pos, end_pos;
    int min_x, min_y;
    int diff_x, diff_y;
    bool can_rubberBanding = true;

    while (end_index <= path_reverse.size() - 1)
    {
        can_rubberBanding = true;
        start_pos = path_reverse[start_index];
        end_pos = path_reverse[end_index];

        min_x = std::min(start_pos.col, end_pos.col);
        min_y = std::min(start_pos.row, end_pos.row);
        diff_x = abs(start_pos.col - end_pos.col) + min_x;
        diff_y = abs(start_pos.row - end_pos.row) + min_y;

        for (int i = min_x; i <= diff_x; ++i)
        {
            for (int j = min_y; j <= diff_y; ++j)
            {
                //check if there is wall between start node and end node
                if (terrain->is_wall({ j,i }))
                {
                    ++start_index;
                    ++end_index;
                    can_rubberBanding = false;
                    break;
                }
            }

            if (can_rubberBanding == false)
            {
                break;
            }
        }

        //if can rubber banding is true, erase mid node
        if (can_rubberBanding)
        {
            path_reverse.erase(path_reverse.begin() + end_index - 1);
        }
    }
}

void AStarPather::Smoothing(PathRequest& request)
{
    int size = request.path.size();

    WaypointList::iterator p1 = request.path.begin();
    WaypointList::iterator p2 = p1;

    WaypointList::iterator p3 = p2;
    p3++;
    WaypointList::iterator p4 = p3;

    if (size >= 3)
    {
        p4++;
    }

    std::vector<Vec3> output_points;
    WaypointList::iterator insert_it = p3;
    WaypointList::iterator end_it = request.path.end();
    end_it--;

    while (insert_it != request.path.end())
    {
        output_points.clear();
        output_points.push_back(Catmull_Rom_Spline(*p1, *p2, *p3, *p4, 0.25f));
        output_points.push_back(Catmull_Rom_Spline(*p1, *p2, *p3, *p4, 0.5f));
        output_points.push_back(Catmull_Rom_Spline(*p1, *p2, *p3, *p4, 0.75f));

        request.path.insert(insert_it, output_points.begin(), output_points.end());
        insert_it++;

        if (p1 != p2)
        {
            p1 = p2;
        }
        p2 = p3;

        if (p3 != end_it)
        {
            p3 = p4;
        }
        if (p4 != end_it)
        {
            ++p4;
        }
    }

}

Vec3 AStarPather::Catmull_Rom_Spline(Vec3 point_1, Vec3 point_2, Vec3 point_3, Vec3 point_4, float s)
{
    Vec3 output_point;
    float three_times = std::pow(s, 3);
    float two_times = std::pow(s, 2);

    output_point = point_1 * (-0.5 * three_times + two_times - 0.5 * s) + point_2 * (1.5 * three_times - 2.5 * two_times + 1.0) +
        point_3 * (-1.5 * three_times + 2.0 * two_times + 0.5 * s) + point_4 * (0.5 * three_times - 0.5 * two_times);

    return output_point;
}

void AStarPather::UpdatePath(PathRequest& request)
{
    int size = path_reverse.size();
    Vec3 curr_vec_pos;

    for (int i = size - 1; i >= 0; --i)
    {
        curr_vec_pos = terrain->get_world_position(path_reverse[i]);
        request.path.push_back(curr_vec_pos);
    }
}

void AStarPather::AddPointsBack(PathRequest& request)
{
    int size = path_reverse.size();
    int index = 0;
    float distance = 0.f;
    GridPos add_pos;
    GridPos min_pos;

    GridPos point_1_grid;
    GridPos point_2_grid;

    Vec3 point_1;
    Vec3 point_2;
    Vec3 new_point;

    WaypointList::iterator path_start = request.path.begin();
    WaypointList::iterator path_end = path_start;

    ++path_end;

    while (path_end != request.path.end())
    {
        point_1 = *path_start;
        point_2 = *path_end;

        point_1_grid = terrain->get_grid_position(point_1);
        point_2_grid = terrain->get_grid_position(point_2);
        distance = DistanceBetweenTwoGridPos(point_1_grid, point_2_grid);

        if(distance > 1.5f)
        {
            new_point.x = abs(point_1.x - point_2.x) / 2.f + std::min(point_1.x, point_2.x);
            new_point.y = point_1.y;
            new_point.z = abs(point_1.z - point_2.z) / 2.f + std::min(point_1.z, point_2.z);

            request.path.insert(path_end, new_point);
            --path_end;
        }
        else
        {
            ++path_start;
            ++path_end;
        }
    }
}

float AStarPather::DistanceBetweenTwoGridPos(GridPos pos_1, GridPos pos_2)
{
    int diff_x = pos_1.col - pos_2.col;
    int diff_y = pos_1.row - pos_2.row;

    float distance = std::sqrt(diff_x * diff_x + diff_y * diff_y);

    return distance;
}

void AStarPather::InsertNodeOnOpenList(Node new_node)
{
    int size = openList.size();

    for (std::vector<Node>::iterator it = openList.begin(); it != openList.end(); ++it)
    {
        if ((*it).cost >= new_node.cost)
        {
            openList.insert(it, new_node);
            break;
        }
    }

    //if openlist is empty, then push back the node.
    if (openList.size() == size)
    {
        openList.push_back(new_node);
    }
}

void AStarPather::UpdatePathFromCloseList(PathRequest& request)
{
    int size = closeList.size() - 1;
    Node curr_node;
    GridPos curr_grid_pos;
    int curr_parent_pos;

    curr_grid_pos = terrain->get_grid_position(request.goal);
    curr_parent_pos = curr_grid_pos.col + curr_grid_pos.row * width;

    for (int i = size; i >= 0; --i)
    {
        curr_node = closeList[i];
        if (curr_node.pos == curr_parent_pos)
        {
            curr_grid_pos = GetGridPosFromPosition(curr_node.pos);
            curr_parent_pos = curr_node.pos_parent;
            path_reverse.push_back(curr_grid_pos);
        }
    }
}

PathResult AStarPather::compute_path(PathRequest& request)
{
    /*
        This is where you handle pathing requests, each request has several fields:

        start/goal - start and goal world positions
        path - where you will build the path upon completion, path should be
            start to goal, not goal to start
        heuristic - which heuristic calculation to use
        weight - the heuristic weight to be applied
        newRequest - whether this is the first request for this path, should generally
            be true, unless single step is on

        smoothing - whether to apply smoothing to the path
        rubberBanding - whether to apply rubber banding
        singleStep - whether to perform only a single A* step
        debugColoring - whether to color the grid based on the A* state:
            closed list nodes - yellow
            open list nodes - blue

            use terrain->set_color(row, col, Colors::YourColor);
            also it can be helpful to temporarily use other colors for specific states
            when you are testing your algorithms

        method - which algorithm to use: A*, Floyd-Warshall, JPS+, or goal bounding,
            will be A* generally, unless you implement extra credit features

        The return values are:
            PROCESSING - a path hasn't been found yet, should only be returned in
                single step mode until a path is found
            COMPLETE - a path to the goal was found and has been built in request.path
            IMPOSSIBLE - a path from start to goal does not exist, do not add start position to path
    */

    // WRITE YOUR CODE HERE
    GridPos empty{ 0, 0 };
    GridPos parent_pos;
    Node parent_node;
    Node curr_node;

    if (request.newRequest)
    {
        width = terrain->get_map_width();
        height = terrain->get_map_height();

        openList.clear();
        closeList.clear();
        listType_InBoard.clear();
        path_reverse.clear();

        start = terrain->get_grid_position(request.start);
        goal = terrain->get_grid_position(request.goal);
        listType_InBoard = std::vector<int>(width * height, NONELIST);

        if (request.settings.debugColoring)
        {
            terrain->set_color(start, Colors::Orange);
            terrain->set_color(goal, Colors::Orange);
        }
        
        float heuristic = CalculateHeuristic(start, goal, request.settings.heuristic) * request.settings.weight;
        openList.push_back(NodeInitialize(empty, start, heuristic, 0.f));

        listType_InBoard[openList[0].pos] = OPENLIST;
    }

    int goal_pos = goal.col + goal.row * width;
    
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

    return PathResult::COMPLETE;
}

```
</div></details>
