# Liaoning Ship’s Voyage

Liaoning ship, which named after a province of China, is the first aircraft carrier commissioned into the People's Liberation Army Navy. It was bought from Ukraine as a stripped hulk and was rebuilt by China as an important part of China’s blue water Navy plan. Liaoning ship has sailed far into the Pacific Ocean for serval times, which shows the power and resolve of China to defend her integrity of territory.

Now Liaoning ship is on a new voyage to the Atlantic Ocean for a maneuver! The vast maneuver region on the ocean can be seen as an n×n grid which has n×n crosspoints. Each crosspoint stands for a check point of the maneuver region. Liaoning starts from the bottom-left check point whose coordinate is (0,0), and its destination is the upper-right checkpoint whose coordinate is (n-1,n-1). The positive side of the x axis points to the right, and the positive side of the y axis points up. All check points’ coordinates are integral. During each move, Liaoning can go from one check point to its adjacent 8 check points along a straight line, and each move takes Liaoning one day. Some check points are not available to go due to the bad weather. And, as you know, on the Atlantic Ocean, there is a Bermuda Triangle in which many ships and planes were missing. Liaoning can’t take risk to go into that triangle. Of course, Liaoning can’t go outside the maneuver region. Please figure out a route for Liaoning to reach its destination as soon as possible.

## 分析

实际上本体是个Dijkstra，但是两点是否相连需要使用到计算几何的东西。重点就在于如何判断行进过程是否穿过了百慕大三角。

<!--more-->

然后就WA了8发。

实际判断时会有很多问题。例如从边上经过，从外界到边，从边到边，从边到外界，从外界到点，从点到外界，从边穿过点……如果一开始选择的判断方法有锅的话，之后就改不出来了，只能WA……

最终想出来的解决办法是拿到行进线段与三角形三边的交点，连接交点构造新线段后，判断线段中点是否在三角形内；并同时判断起点终点是否在三角形内。这样就能避免掉复杂的讨论。

其次，顺便把已经在三角形内的整点全部标记了不能访问。

这样就过了。

## 代码

板子判点在线段部分有问题，要改。

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include <string>
#include <cstring>
#include <stack>
#include <queue>
#include <cmath>
#include <cstdlib>
#include <queue>
using namespace std;
const int MAXN=30;


bool fuck=false;
const double EPS=1e-7;
const double PI=acos(-1);
int sgn(double x){
    return (x>-EPS)-(x<EPS);
}
struct Vec{
    double x,y;//never change it yourself unless you dont need polar angle.
    double _polar;// make cache to accumulate speed as atan is too slow.

    Vec(){
        x=y=0;
    }
    Vec(double x,double y):x(x),y(y){
        _polar=atan2(y,x);
    }
    double dot(const Vec &b)const{
        return x*b.x+y*b.y;
    }
    double cross(const Vec &b)const{
        return x*b.y-b.x*y;
    }
    double len(){
        return sqrt(sqlen());
    }
    double sqlen(){
        return x*x+y*y;
    }
    Vec normalize(){
        double l=len();
        return Vec(x/l,y/l);
    }
    Vec rotate(double angle){
        return Vec(x*cos(angle)-y*sin(angle),x*sin(angle)+y*cos(angle));
    }
    Vec operator * (double factor)const{
        return Vec(x*factor,y*factor);
    }
    double operator * (const Vec &b)const{
        return cross(b);
    }
    Vec operator - (const Vec &b)const{
        return Vec(x-b.x,y-b.y);
    }
    Vec operator +(const Vec &b)const{
        return Vec(x+b.x,y+b.y);
    }
    double polar()const{
        return _polar;
    }
    bool leftby(const Vec &b)const{
        return sgn(b.cross(*this))>0;
    }
    bool samed(const Vec &b)const{
        return sgn(this->cross(b))==0 && sgn(this->dot(b))>0;
    }
    bool operator<(const Vec &b)const{
        return this->polar()<b.polar();
    }
    
    bool operator==(const Vec &b)const{
		return sgn(b.x-this->x)==0 &&sgn(b.y-this->y)==0;
	}
};
ostream& operator<<(ostream& out,const Vec &b){
    out<<"("<<b.x<<","<<b.y<<")";
    return out;
}
typedef Vec Point;
struct Line{
    Point pos;
    Vec dirc;
    Line(Point pos=Point(0,0),Vec dirc=Vec(0,0)):pos(pos),dirc(dirc){}
    static Line fromPoints(Point a,Point b){
        return Line(a,b-a);
    }
    double getarea(const Line &b)const{
        return abs(dirc.cross(b.dirc));
    }
    // 获得垂线
    Line getppd(){
        return Line(pos+dirc*0.5,dirc.rotate(PI/2));
    }

    //TODO: what will happen if they have no intersection,-nan
    Point getintersection(const Line &b)const{
        Vec down=this->pos-b.pos;
        double aa=b.dirc.cross(down);
        double bb=this->dirc.cross(b.dirc);
        return this->pos+this->dirc*(aa/bb);
    }

    bool point_on_line(Point point){
        if(!dirc.samed(point-pos))
            return false;
        if(sgn((point-pos).sqlen()-dirc.sqlen())>0)
            return false;
        return true;
    }

    bool on_ver(Point point){
        if(sgn(point.x-pos.x)==0 && sgn(point.y-pos.y)==0)return true;
        if(sgn(point.x-(pos.x+dirc.x))==0 && sgn(point.y-(pos.y+dirc.y))==0)return true;
        return false;
    }

    double get_distance(Point point){
        Line ppd=getppd();
        ppd.pos=point;

        Point intersection=getintersection(ppd);

        ppd.dirc=intersection-point;
        Vec v=intersection-pos;

        return abs(v.cross(point-pos)/v.len());
    }

    double get_distance(Line line){
        return get_distance(line.pos);
    }
};

Point points[3];
Line triangle[3];

char game[MAXN][MAXN];

double get_area(vector<Point> &points){
    if(points.size()<3){
        return -1;
    }
    sort(points.begin(),points.end(),[](const Point &a,const Point &b){
        return a.polar()<b.polar();
    });
    Point base(0,0);
    Point last=points[0];
    double res=0;
    for(int i=1;i<points.size();i++){
        Vec a=last-base,b=points[i]-base;
        res+=a.cross(b)/2;

        last=points[i];
    }
    //add the last point(also the first point)
    Vec a=last-base,b=points[0]-base;
    res+=a.cross(b)/2;

    return fabs(res);
}

bool in_triangle(Point p){
    if(fuck)return false;

    for(int i=0;i<3;i++){
        if(Line::fromPoints(points[i%3],points[(i+1)%3]).point_on_line(p))return false;
    }

    double areas=0;
    for(int i=0;i<3;i++){
        vector<Point> ps;
        ps.push_back(points[i%3]);
        ps.push_back(points[(i+1)%3]);
        ps.push_back(p);
        areas+=get_area(ps);
    }

    vector<Point> ps;
    for(int i=0;i<3;i++)ps.push_back(points[i]);
    return sgn(areas-get_area(ps))==0;
}

bool has_intersetct(Line line){
    if(fuck)return false;
    int cnt=0; 
    Point p[3];
    bool flg[3]={0,0,0};
    for(int i=0;i<3;i++){
        if(sgn(line.dirc.cross(triangle[i].dirc))==0)continue;
        p[i]=line.getintersection(triangle[i]);
        if(line.point_on_line(p[i])||p[i]==line.pos) flg[i]=1;
    }
    if(in_triangle(line.pos)) return true;
    if(in_triangle(line.pos+line.dirc)) return true;
    Line l;
    if(flg[0]&&flg[1])
    {
		l=Line::fromPoints(p[0],p[1]);
	    if(in_triangle(l.pos+l.dirc*0.5)) return true;
    }
    if(flg[2]&&flg[1])
    {
		l=Line::fromPoints(p[2],p[1]);
	    if(in_triangle(l.pos+l.dirc*0.5)) return true;
    }
    if(flg[0]&&flg[2])
    {
		l=Line::fromPoints(p[0],p[2]);
	    if(in_triangle(l.pos+l.dirc*0.5)) return true;
    }
    return false;
}

struct v4q{
    int i,j,step;
    v4q(int i,int j,int step):i(i),j(j),step(step){}

    bool operator<(const v4q &other)const{
        return step>other.step;
    }
};
int nlen;
int dist[MAXN][MAXN];
bool vis[MAXN][MAXN];
int solve(){
    priority_queue<v4q> pq;
    pq.push(v4q(1,1,0));
    memset(dist,0x3f,sizeof(dist));
    memset(vis,0,sizeof(vis));
    dist[1][1]=0;
    while(!pq.empty()){
        v4q cur=pq.top();pq.pop();

        if(vis[cur.i][cur.j])continue;
        vis[cur.i][cur.j]=1;
		//printf("now:%d %d\n",cur.i,cur.j);
        for(int i=max(1,cur.i-1);i<=min(nlen,cur.i+1);i++){
            for(int j=max(1,cur.j-1);j<=min(nlen,cur.j+1);j++){
                if(i==cur.i && j==cur.j)continue;
                if(game[i][j]==&#039;#&#039;)continue;
                if(has_intersetct(Line::fromPoints(Point(j-1,i-1),Point(cur.j-1,cur.i-1))))continue;
               //cout<<"arrive "<<i<<","<<j;
                if(dist[i][j]>dist[cur.i][cur.j]+1){
                    dist[i][j]=dist[cur.i][cur.j]+1;
                    //cout<<" and update it to "<<dist[i][j];
                    pq.push(v4q(i,j,dist[i][j]));
                }
                //cout<<endl;
            }
        }
    }
    return dist[nlen][nlen];
}


int main(){
//	freopen("in.txt","r",stdin);
//	freopen("out.txt","w",stdout);
    while(cin>>nlen){
    	fuck=0;
        for(int i=0;i<3;i++){
            Point &p=points[i];
            cin>>p.x>>p.y;
        }

        for(int i=0;i<3;i++){
            triangle[i]=Line::fromPoints(points[i%3],points[(i+1)%3]);
        }

        for(int i=0;i<3;i++){
            if(sgn(triangle[i%3].dirc.cross(triangle[(i+1)%3].dirc))==0){
                fuck=1;
            }
        }

        for(int i=nlen;i>=1;i--){
            for(int j=1;j<=nlen;j++){
                cin>>game[i][j];
            }
        }

        //solve triangle
        
        for(int i=1;i<=nlen;i++){
            for(int j=1;j<=nlen;j++){
                if(in_triangle(Point(j-1,i-1)))game[i][j]=&#039;#&#039;;
            }
        }
        
        /*
        for(int i=nlen;i>=1;i--){
            for(int j=1;j<=nlen;j++){
                cout<<game[i][j];
            }
            cout<<endl;
        }
        */
        
        

        int ans=solve();
        if(ans!=0x3f3f3f3f)cout<<ans<<"\n";
        else cout<<"-1\n";
    }

    return 0;
}
```

