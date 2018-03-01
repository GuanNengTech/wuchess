# wuchess
简易的五子棋对战平台

![](https://github.com/GuanNengTech/wuchess/blob/master/wuchessbg.png)

## 介绍

简单的五子棋对战平台，同屏对战。画面没有任何华丽的地方却简洁大方。有不足之处,欢迎大家提供建议和指点，予人玫瑰，手有余香。

### 实现思路及主要代码详解

这个项目决定开源出来，招募感兴趣的程序猿一起加入进来，目标是制作一个不含广告、简洁大方、棋类丰富的对战平台。

##### 1.绘制棋盘

  利用Quartz2D绘制棋盘.代码如下

	- (void)drawBackground:(CGSize)size{
	    
	    self.gridWidth = (size.width - 2 * kBoardSpace) / self.gridCount;
	    
	    //1.开启图像上下文
	    UIGraphicsBeginImageContext(size);
	    //2.获取上下文
	    CGContextRef ctx = UIGraphicsGetCurrentContext();
	    
	    CGContextSetLineWidth(ctx, 0.8f);
	    //3.1 画16条竖线
	    for (int i = 0; i <= self.gridCount; i ++) {
	        CGContextMoveToPoint(ctx, kBoardSpace + i * self.gridWidth , kBoardSpace);
	        CGContextAddLineToPoint(ctx, kBoardSpace + i * self.gridWidth , kBoardSpace + self.gridCount * self.gridWidth);
	    }
	    //3.1 画16条横线
	    for (int i = 0; i <= self.gridCount; i ++) {
	        CGContextMoveToPoint(ctx, kBoardSpace, kBoardSpace  + i * self.gridWidth );
	        CGContextAddLineToPoint(ctx, kBoardSpace + self.gridCount * self.gridWidth , kBoardSpace + i * self.gridWidth);
	    }
	    CGContextStrokePath(ctx);
	    
	    //4.获取生成的图片
	    UIImage *image=UIGraphicsGetImageFromCurrentImageContext();
	    //5.显示生成的图片到imageview
	    UIImageView * imageView = [[UIImageView alloc]initWithImage:image];
	    [self addSubview:imageView];
	    UIGraphicsEndImageContext();
	}
##### 2.点击棋盘落子

1.根据落子位置求出该棋子的行号与列号.
2.判断落子位置是否已经有棋子,有则不能下.如果没有,将棋子保存在字典中,以列号和行号组合起来的字符串为key值.
  代码如下

	//点击棋盘,下棋
	- (void)tapBoard:(UITapGestureRecognizer *)tap{
	
	    CGPoint point = [tap locationInView:tap.view];
	    //计算下子的列号行号
	    NSInteger col = (point.x - kBoardSpace + 0.5 * self.gridWidth) / self.gridWidth;
	    NSInteger row = (point.y - kBoardSpace + 0.5 * self.gridWidth) / self.gridWidth;
	    NSString * key = [NSString stringWithFormat:@"%ld-%ld",col,row];
	    if (![self.chessmanDict.allKeys containsObject:key]) {
	        UIView * chessman = [self chessman];
	        chessman.center = CGPointMake(kBoardSpace + col * self.gridWidth, kBoardSpace + row * self.gridWidth);
	        [self addSubview:chessman];
	        [self.chessmanDict setValue:chessman forKey:key];
	        self.lastKey = key;
	        //检查游戏结果
	        [self checkResult:col andRow:row andColor:self.isBlack];
	        self.isBlack = !self.isBlack;
	    }
	}

##### 3.检测游戏结果

每落一个棋子就要多游戏结果进行一次检查,判断四个方向上是否有大于等于5个同色的棋子连成一线,有则提示游戏输赢结果,无则游戏继续.算法为,从当前棋子位置向前遍历,直到遇到与自己不同色的棋子,累加同色棋子的个数,再往后遍历,直到遇到与自己不同色的棋子,累加同色棋子的个数.得到该方向相连同色棋子的总个数
代码如下

	//判断是否大于等于五个同色相连
	- (BOOL)checkResult:(NSInteger)col andRow:(NSInteger)row andColor:(BOOL)isBlack andDirection:(GmkDirection)direction{
	
	    if (self.sameChessmanArray.count >= 5) {
	        return YES;
	    }
	    UIColor * currentChessmanColor = [self.chessmanDict[[NSString stringWithFormat:@"%ld-%ld",col,row]] backgroundColor];
	    [self.sameChessmanArray addObject:self.chessmanDict[self.lastKey]];
	    switch (direction) {
	        //水平方向检查结果
	        case GmkHorizontal:{
	            //向前遍历
	            for (NSInteger i = col - 1; i > 0; i --) {
	                NSString * key = [NSString stringWithFormat:@"%ld-%ld",i,row];
	                if (![self.chessmanDict.allKeys containsObject:key] || [self.chessmanDict[key] backgroundColor] != currentChessmanColor) break;
	                [self.sameChessmanArray addObject:self.chessmanDict[key]];
	            }
	            //向后遍历
	            for (NSInteger i = col + 1; i < kGridCount; i ++) {
	                NSString * key = [NSString stringWithFormat:@"%ld-%ld",i,row];
	                if (![self.chessmanDict.allKeys containsObject:key] || [self.chessmanDict[key] backgroundColor] != currentChessmanColor) break;
	                [self.sameChessmanArray addObject:self.chessmanDict[key]];
	            }
	            if (self.sameChessmanArray.count >= 5) {
	                [self alertResult];
	                return YES;
	            }
	            [self.sameChessmanArray removeAllObjects];
	            
	        }
	            break;
	        case GmkVertical:{
	            //向前遍历
	            for (NSInteger i = row - 1; i > 0; i --) {
	                NSString * key = [NSString stringWithFormat:@"%ld-%ld",col,i];
	                if (![self.chessmanDict.allKeys containsObject:key] || [self.chessmanDict[key] backgroundColor] != currentChessmanColor) break;
	                [self.sameChessmanArray addObject:self.chessmanDict[key]];
	            }
	            //向后遍历
	            for (NSInteger i = row + 1; i < kGridCount; i ++) {
	                NSString * key = [NSString stringWithFormat:@"%ld-%ld",col,i];
	                if (![self.chessmanDict.allKeys containsObject:key] || [self.chessmanDict[key] backgroundColor] != currentChessmanColor) break;
	                [self.sameChessmanArray addObject:self.chessmanDict[key]];
	            }
	            if (self.sameChessmanArray.count >= 5) {
	                [self alertResult];
	                return YES;
	            }
	            [self.sameChessmanArray removeAllObjects];
	            
	        }
	            break;
	        case GmkObliqueDown:{
	            
	            //向前遍历
	            NSInteger j = col - 1;
	            for (NSInteger i = row - 1; i >= 0; i--,j--) {
	                NSString * key = [NSString stringWithFormat:@"%ld-%ld",j,i];
	                if (![self.chessmanDict.allKeys containsObject:key] || [self.chessmanDict[key] backgroundColor] != currentChessmanColor || j < 0) break;
	                [self.sameChessmanArray addObject:self.chessmanDict[key]];
	            }
	            //向后遍历
	            j = col + 1;
	            for (NSInteger i = row + 1 ; i < kGridCount; i++,j++) {
	                NSString * key = [NSString stringWithFormat:@"%ld-%ld",j,i];
	                if (![self.chessmanDict.allKeys containsObject:key] || [self.chessmanDict[key] backgroundColor] != currentChessmanColor || j > kGridCount) break;
	                [self.sameChessmanArray addObject:self.chessmanDict[key]];
	            }
	            if (self.sameChessmanArray.count >= 5) {
	                [self alertResult];
	                return YES;
	            }
	            [self.sameChessmanArray removeAllObjects];
	            
	        
	        }
	            break;
	        case GmkObliqueUp:{
	            //向前遍历
	            NSInteger j = col + 1;
	            for (NSInteger i = row - 1; i >= 0; i--,j++) {
	                NSString * key = [NSString stringWithFormat:@"%ld-%ld",j,i];
	                if (![self.chessmanDict.allKeys containsObject:key] || [self.chessmanDict[key] backgroundColor] != currentChessmanColor || j > kGridCount) break;
	                [self.sameChessmanArray addObject:self.chessmanDict[key]];
	            }
	            //向后遍历
	            j = col - 1;
	            for (NSInteger i = row + 1 ; i < kGridCount; i++,j--) {
	                NSString * key = [NSString stringWithFormat:@"%ld-%ld",j,i];
	                if (![self.chessmanDict.allKeys containsObject:key] || [self.chessmanDict[key] backgroundColor] != currentChessmanColor || j < 0) break;
	                [self.sameChessmanArray addObject:self.chessmanDict[key]];
	            }
	            if (self.sameChessmanArray.count >= 5) {
	                [self alertResult];
	                return YES;
	            }
	            [self.sameChessmanArray removeAllObjects];
	            
	        }
	            break;
	    }
	    return NO;
	}

##### 4.对外提供,重新开始,悔棋,切换初高级棋盘的三个接口


重新开始
	- (void)newGame{
	    
	    self.isOver = NO;
	    self.lastKey = nil;
	    [self.sameChessmanArray removeAllObjects];
	    self.userInteractionEnabled = YES;
	    [self.chessmanDict removeAllObjects];
	    for (UIView * view in self.subviews) {
	        if ([view isKindOfClass:[UIImageView class]]) {
	            continue;
	        }
	        [view removeFromSuperview];
	    }
	    self.isBlack = NO;
	}

悔棋

      //撤回至上一步棋
	- (void)backOneStep:(UIButton *)sender{
	
	    if(self.isOver) return;
	    
	    if (self.lastKey == nil) {
	        sender.enabled = NO;
	        CGFloat width = SCREEN_WIDTH * 0.4 * SCREEN_WIDTH_RATIO;
	        UIView * tip = [[UIView alloc]initWithFrame:CGRectMake(0, 0, width, 0.6 * width)];
	        tip.backgroundColor = [UIColor colorWithWhite:1 alpha:0.8];
	        tip.layer.cornerRadius = 8.0f;
	        [self addSubview:tip];
	        tip.center = CGPointMake(self.width * 0.5, self.height * 0.5);
	        UILabel * label = [[UILabel alloc]init];
	        label.text = self.chessmanDict.count > 0 ? @"只能悔一步棋!!!" : @"请先落子!!!";
	        label.font = [UIFont systemFontOfSize:15];
	        [label sizeToFit];
	        label.center = CGPointMake(tip.width * 0.5, tip.height * 0.5);
	        [tip addSubview:label];
	        
	        self.userInteractionEnabled = NO;
	        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
	            self.userInteractionEnabled = YES;
	            sender.enabled = YES;
	            [tip removeFromSuperview];
	            
	        });
	        return;
	    }
	    [self.chessmanDict removeObjectForKey:self.lastKey];
	    [self.subviews.lastObject removeFromSuperview];
	    self.isBlack = !self.isBlack;
	    self.lastKey = nil;
	}

切换初高级键盘

      //改变键盘级别
	- (void)changeBoardLevel{
	    
	    for (UIView * view in self.subviews) {
	        [view removeFromSuperview];
	    }
	    [self newGame];
	    self.isHighLevel = !self.isHighLevel;
	    [self drawBackground:self.bounds.size];
	}
  
#### 一个小技巧

用字典存放棋子,以棋子的列号和行号组合起来的字符串为key值,value值为棋子view.这样处理,在判断某行某列是否有棋子就非常简单了.
