# SodokuSolver
```c++
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
#define MAX_LENGTH 9

int soduku[MAX_LENGTH][MAX_LENGTH] = {0};
// int soduku[MAX_LENGTH][MAX_LENGTH] = {3,9,0,6,0,7,5,1,0,
//     5,0,8,9,0,2,0,4,3,
//     7,0,1,0,0,8,0,0,9,
//     8,1,0,4,6,0,3,9,2,
//     6,3,0,1,2,9,0,7,4,
//     0,2,9,0,7,0,6,5,0,
//     0,5,3,0,0,0,4,8,0,
//     1,0,4,0,0,6,0,0,7,
//     0,0,6,0,8,4,0,2,5};
//Record the slots which need to fill
vector<int>row_list[MAX_LENGTH];
vector<int>col_list[MAX_LENGTH];

//record possible answers.
vector<int>row_vec[MAX_LENGTH];
vector<int>col_vec[MAX_LENGTH];
vector<int>box_vec[MAX_LENGTH];

//Get the corresponding box index.
int getBoxIndex(int _i,int _j)
{
    int i = _i / 3;
    int j = _j / 3;
    return 3 * i + j;
}
//record which slot need to fill.
int getSlotsToFill(vector<int>& row_list,vector<int>& col_list)
{
    int num = 0;
    row_list.clear();
    col_list.clear();
    for(int i = 0;i < MAX_LENGTH;++i)
        for(int j = 0;j < MAX_LENGTH;++j)
        {
            if(soduku[i][j] == 0)
            {
                num++;
                row_list.push_back(i);
                col_list.push_back(j);
            }
        }
    return num;
}

void printSoduku(){
    
    for(int i = 0;i < MAX_LENGTH;++i)
    {
        for(int j = 0; j < MAX_LENGTH; j++)
        {
            cout<< soduku[i][j];
        }
        cout <<endl;
    }
}

int checkAndGetLikelyAns()
{
    //row
    for(int i = 0;i < MAX_LENGTH;++i)
    {
        bool record[MAX_LENGTH] = {false};
        for(int j = 0; j < MAX_LENGTH; j++)
        {
            int val = soduku[i][j] - 1;
            if(val < 0)//empty slot
                continue;
            if(record[val])//invalid soduku
                return -1;
            record[val] = true;
        }
        for(int j = 0;j < MAX_LENGTH;++j)
        {
            if(!record[j])
                row_vec[i].push_back(j+1);
        }
    }

    //col
    for(int j = 0;j < MAX_LENGTH;++j)
    {
        bool record[MAX_LENGTH] = {false};
        for(int i = 0;i < MAX_LENGTH;++i)
        {
            int val = soduku[i][j] - 1;
            if(val < 0)
                continue;
            if(record[val])
                return -1;
            record[val] = true;
        }
        for(int i = 0;i < MAX_LENGTH;++i)
        {
            if(!record[i])
                col_vec[j].push_back(i+1);
        }
    }
    
    //block
    for(int block = 0;block < MAX_LENGTH;block++)
    {
        bool record[MAX_LENGTH] = {false};
        int row_delta = (block / 3) * 3;
        int col_delta = (block % 3) * 3;
        //for each box
        for(int i = 0;i < 3;++i)
            for(int j = 0;j < 3;++j)
            {
                int val = soduku[i+row_delta][j+col_delta] - 1;
                if(val < 0)continue;
                if(record[val])
                    return -1;
                record[val] = true;
            }
        for(int i = 0;i < MAX_LENGTH;++i)
        {
            if(!record[i])
                box_vec[block].push_back(i+1);
        }
    }
    return 0;
}

//Find intersection 
vector<int>findInterSection(vector<int>&rol,vector<int>&col,vector<int>&block)
{
    vector<int>result;
    bool r[MAX_LENGTH] = {false};
    bool c[MAX_LENGTH] = {false};
    bool b[MAX_LENGTH] = {false};
    for(int i = 0;i < rol.size();++i)
    {
       // cout <<"heehhehe\n";
        r[rol[i] - 1] = true;
    }
    for(int i = 0;i < col.size();++i)
    {
        c[col[i] - 1] = true;
    }
    for(int i = 0;i < block.size();++i)
    {
        b[block[i] - 1] = true;
    }
    for(int i = 0;i < MAX_LENGTH;++i)
    {
        if(r[i] && c[i] && b[i])
        {
            result.push_back(i+1);
        }
    }
    return result;
}

//kernel
bool flag = false;
void runKernel(int n, int ans_slots,vector<int>&row_list,vector<int>&col_list)
{
    if(flag)return;
    if(n == ans_slots)
    {
       printSoduku();
       flag = true;
        return;
    }
    //get the fucking position
    int row = row_list[n];
    int col = col_list[n];
    int block = getBoxIndex(row,col);
   // cout <<"hahahha\n";
    //get the possible ans
    vector<int>candidates = findInterSection(row_vec[row],col_vec[col],box_vec[block]);
    if(candidates.size() == 0){
        //cout << "Failed\n";
        return;
    }
    int try_ans;
    for(int i = 0;i < candidates.size();++i)
    {
        try_ans = candidates[i];

        auto iter = find(row_vec[row].begin(),row_vec[row].end(),try_ans);
        row_vec[row].erase(iter);

        iter = find(col_vec[col].begin(),col_vec[col].end(),try_ans);
        col_vec[col].erase(iter);

        iter = find(box_vec[block].begin(),box_vec[block].end(),try_ans);
        box_vec[block].erase(iter);
       // cout << "i am here!\n";
        soduku[row][col] = try_ans;
        //recursively call this function
        runKernel(n+1,ans_slots,row_list,col_list);
        //backtracking
        soduku[row][col] = 0;
        row_vec[row].push_back(try_ans);
        col_vec[col].push_back(try_ans);
        box_vec[block].push_back(try_ans);
    }
       
}

int main(){
    char buffer[10];
    int t;
    cin >> t;
    while(t--){
        flag = false;
    for(int i = 0;i < MAX_LENGTH;++i)
    {
        row_vec[i].clear();
        col_vec[i].clear();
        box_vec[i].clear();
    }
    for(int i = 0;i < MAX_LENGTH;++i)
    {
        cin >> buffer;
        for(int j = 0; j < MAX_LENGTH; j++)
        {
            soduku[i][j] = buffer[j] - '0';
        }
        
    }
    
    //printSoduku();
    vector<int> empty_row_list;
    vector<int> empty_col_list;
    int unknown_num = getSlotsToFill(empty_row_list, empty_col_list);
    //cout << unknown_num << endl;
    if(checkAndGetLikelyAns() < 0){
        //cout <<"nonoonononno\n";
        return 0;
    }
    //cout << "*************************\n";
    runKernel(0,unknown_num,empty_row_list,empty_col_list);
    }
    return 0;
}
```
