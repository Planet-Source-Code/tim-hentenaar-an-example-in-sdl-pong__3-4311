<div align="center">

## An example in SDL: Pong


</div>

### Description

This is an example of using basic SDL in C++. From what i've seen on PSC, this appears to be the first piece of SDL code here :P
 
### More Info
 
It's still a little buggy,(e.g. collision isn't correct, there are a few bounce issues...) but i didn't design it to be exact. I designed this to illustrate some of the key functions of SDL.


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Tim Hentenaar](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/tim-hentenaar.md)
**Level**          |Beginner
**User Rating**    |5.0 (10 globes from 2 users)
**Compatibility**  |C, C\+\+ \(general\), Microsoft Visual C\+\+, Borland C\+\+, UNIX C\+\+
**Category**       |[Miscellaneous](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/miscellaneous__3-1.md)
**World**          |[C / C\+\+](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/c-c.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/tim-hentenaar-an-example-in-sdl-pong__3-4311/archive/master.zip)





### Source Code

```
// SDL Pong: A Ping-Pong Demo
// (C) 2002 Matriark TerVel
// Distributed under the GNU General Public License
#include <SDL/SDL.h>
#include "SFont.h"
#include <stdlib.h>
#include <string.h>
#include <math.h>
class object {
	public:
	SDL_Rect rect;
	SDL_Surface *surf;
	unsigned int x_inc;
	unsigned int y_inc;
	double angle;
	int dir;
	int mode;
	double slope;
	~object();
	void Move(int dx, int dy);
	void Collision();
	double GetSlope(SDL_Rect target);
	void Point(int target);
};
long score = 0; // Current score
SDL_Surface *screen; // Our surface
object ball; // the ball
object p_paddle; // the player paddle
SDL_Surface *nums; // font
void Blit(SDL_Surface *src, int x, int y, SDL_Surface *dest, int dx, int dy) {
 SDL_Rect dest_t; // used to hold info about the rect to blit to the dest surface
 SDL_Rect src_t; // used to hold info about the rect to blit from on the source
 dest_t.x = dx;
 dest_t.y = dy;
 dest_t.w = src->w;
 dest_t.h = src->h;
 src_t.x = x;
 src_t.y = y;
 src_t.w = src->w;
 src_t.h = src->h;
 // Blit Surface onto screen
 SDL_BlitSurface(src, &src_t, dest, &dest_t);
 // Update the changed portion of the destination
 SDL_UpdateRect(dest,0,0,0,0);
}
void BlitScore(int scr, SDL_Surface *scrn) {
 char xtxt[10];
 memset(xtxt,'\0',sizeof(xtxt));
 sprintf(xtxt,"Score: %d",scr);
 PutString(scrn,5,5,xtxt);
}
void FreeSurfs() {
 SDL_FreeSurface(nums);
}
void object::Move(int dx, int dy) { // Move the object
	if (!screen) {
	 fprintf(stderr,"object::move() Screen is NULL!\n");
	 exit(1);
	}
	SDL_FillRect(screen,&rect,SDL_MapRGB(surf->format,0,0,0));
	// Bounds checking
	// 650x50 window at the top for status
	if ((rect.y += (dy * y_inc)) < 25 || (rect.y += (dy * y_inc)) > 300) Collision();
	if ((rect.x += (dx * x_inc)) < 5 || (rect.x += (dx * x_inc)) > 640) Collision();
	Blit(surf,0,0,screen,rect.x,rect.y);
}
object::~object() {
  SDL_FreeSurface(surf);
}
void object::Collision() { // Get new coordinates for direction after a collision
  if (rect.x >= p_paddle.rect.x && rect.x <= (p_paddle.rect.x + p_paddle.rect.w)) {
	 Point(1);
//	 ball.rect.x = rand() % 300;
//	 ball.rect.y = rand() % 300;
	 dir = 1;
   Move(0,-1); // up
	 return;
	}
if (rect.x >= (screen->w - rect.w) + 1 || rect.x < screen->w - 10 || rect.y < screen->h + 25 || rect.y > screen->h - (p_paddle.rect.h + 1)) {
  if (dir == 1) {
	 ball.rect.y += 30;
//	 Move(0,-1); // up
	 dir = 3;
	 Move(0,1);
	 return;
	}
	if (dir == 2) {
	 ball.rect.x -= 20;
//	 Move(0,-1); // right
	 dir = 4;
	 Move(1,0);
	 return;
	}
  if (dir == 3) {
	 ball.rect.y += 25;
//	 Move(-1,0); // down
	 dir = 1;
	 Move(0,-1);
	 return;
	}
  if (dir == 4) {
	 ball.rect.x += 15;
//	 Move(1,0); // left
	 dir = 2;
	 Move(-1,0);
	 return;
	}
}
}
void ball_init() {
 // Choose a random direction
 ball.dir = rand() % 4;
}
Uint32 ball_move(Uint32 interval, void *param) {
  if (ball.dir == 1) ball.Move(0,-1); // up
  if (ball.dir == 2) ball.Move(-1,0); // right
  if (ball.dir == 3) ball.Move(0,1); // down
  if (ball.dir == 4) ball.Move(1,0); // left
//  printf("ball = (%d,%d)\n",ball.rect.x,ball.rect.y);
  return interval;
}
void doWin() {
  BlitScore(score,screen);
  PutString(screen,105,5,"You Win!");
}
void object::Point(int target) {
   if (target == 1) {
	 score++;
	 BlitScore(score,screen);
	 if (score == 15) doWin();
	 }
}
double object::GetSlope(SDL_Rect target) {
  return (double)(((double)target.y - rect.y)/(double)(target.x - rect.x));
}
int main(int argc, char **argv) {
	SDL_Event event;
	// Initialize SDL's Video Component
	if (SDL_Init(SDL_INIT_VIDEO | SDL_INIT_TIMER) < 0) {
		fprintf(stderr,"Unable to initialize SDL: %s\n",SDL_GetError());
		exit(1); // Exit the program
	}
	// Call SDL_Quit on exit()
	atexit(SDL_Quit);
	// Set the video mode
	screen = SDL_SetVideoMode(640,480,16,SDL_SWSURFACE);
	if (!screen) {
		fprintf(stderr,"Unable to set Video Mode: %s\n",SDL_GetError());
		exit(1);
	}
	// Set the caption
	SDL_WM_SetCaption("SDL Pong - (C) 2002 Tim Hentenaar - http://xodian.net",NULL);
	// Initialize Font
	nums = SDL_LoadBMP("font.bmp");
	InitFont(nums);
	// Free it at exit
	atexit(FreeSurfs);
	// Initialize the ball
	ball.rect.x = rand()%300;
	ball.rect.y = rand()%300;
	ball.x_inc = 10;
	ball.y_inc = 10;
	ball.surf = SDL_LoadBMP("ball.bmp");
	ball.rect.w = ball.surf->w;
	ball.rect.h = ball.surf->h;
	p_paddle.x_inc = 15;
	p_paddle.y_inc = 1;
	p_paddle.surf = SDL_LoadBMP("paddle.bmp");
	p_paddle.rect.y = (screen->h + (p_paddle.surf->h) - 30);
	p_paddle.rect.h = p_paddle.surf->h;
	p_paddle.rect.w = p_paddle.surf->w;
	p_paddle.rect.x = 150 + (p_paddle.surf->w);
	// Draw the ball to the screen
	Blit(ball.surf,0,0,screen,ball.rect.x,ball.rect.y);
	// Draw the paddles
	Blit(p_paddle.surf,0,0,screen,p_paddle.rect.x,p_paddle.rect.y);
	score = 0;
	BlitScore(score,screen);
	ball_init();
	Uint8 *keys;
	SDL_TimerID ball_move_timer;
while(true) {
 while (SDL_PollEvent(&event)) {
  if (event.type == SDL_QUIT) exit(0);
  if (event.type == SDL_KEYDOWN) {
   if (event.key.keysym.sym == SDLK_ESCAPE) {
   SDL_RemoveTimer(ball_move_timer);
	 exit(0);
   }
  }
 }
 keys = SDL_GetKeyState(NULL);
 if ( keys[SDLK_LEFT] ) { p_paddle.Move(-1,0); }
 if ( keys[SDLK_RIGHT] ) { p_paddle.Move(1,0); }
 if ( keys[SDLK_s] ) ball_move_timer = SDL_AddTimer(200,ball_move,NULL);
 Blit(p_paddle.surf,0,0,screen,p_paddle.rect.x,p_paddle.rect.y);
}
return 0;
}
```

