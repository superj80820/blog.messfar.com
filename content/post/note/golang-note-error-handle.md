+++
title = 'Golang 錯誤處理'
date = 2024-02-02T00:00:00+08:00
tags = ['go']
+++

- 值得參考的文章:
    - [https://earthly.dev/blog/golang-errors/](https://earthly.dev/blog/golang-errors/)
    - [https://levelup.gitconnected.com/go-error-best-practice-f0864c5c2385](https://levelup.gitconnected.com/go-error-best-practice-f0864c5c2385)
    - [https://www.joeshaw.org/error-handling-in-go-http-applications/](https://www.joeshaw.org/error-handling-in-go-http-applications/)
- recover
    - gin中如果使用goroutine這類async邏輯，必須補充recover的邏輯，才可捕捉錯誤避免崩潰。非async邏輯可直接用`r.Use(gin.Recovery())`捕捉
        - panic發生在非async邏輯中: 如果使用`r.Use(gin.Recovery())`，發生panic會捕捉到此錯誤，並回傳500給呼叫端
            
            ```go
            package main
            
            import (
            	"github.com/gin-gonic/gin"
            )
            
            func main() {
            	r := gin.New()
            	r.Use(gin.Logger())
            	r.Use(gin.Recovery())
            
            	r.GET("/ping", func(c *gin.Context) {
            		func() {
            			panic("panic")
            		}()
            
            		c.JSON(200, gin.H{
            			"message": "pong",
            		})
            	})
            
            	r.Run()
            }
            ```
            
        - panice發生在async邏輯中: 雖使用`r.Use(gin.Recovery())`，但panic發生在goroutine中，則panic不會捕捉，並且程序會崩潰，必須在goroutine補充recover的邏輯，才可捕捉錯誤避免崩潰
            
            ```go
            package main
            
            import (
            	"fmt"
            	"sync"
            
            	"github.com/gin-gonic/gin"
            )
            
            type WaitGroupWithError struct {
            	sync.WaitGroup
            	Error error
            }
            
            func (w *WaitGroupWithError) Wait() error {
            	w.WaitGroup.Wait()
            	return w.Error
            }
            
            func main() {
            	r := gin.New()
            	r.Use(gin.Logger())
            	r.Use(gin.Recovery())
            
            	r.GET("/ping", func(c *gin.Context) {
            		var wg WaitGroupWithError
            		wg.Add(1)
            		go func() {
            			defer func() {
            				// 需要在此透過recover捕捉
            				if err := recover(); err != nil {
            					wg.Error = fmt.Errorf("get panic: %v", err)
            				}
            				wg.Done()
            			}()
            			panic("panic")
            		}()
            		if err := wg.Wait(); err != nil {
            			fmt.Printf("get err: %+v", err)
            			c.JSON(500, gin.H{
            				"message": "internal error",
            			})
            			return
            		}
            
            		c.JSON(200, gin.H{
            			"message": "pong",
            		})
            	})
            
            	r.Run()
            }
            ```
            
        - 參考:
            - [https://segmentfault.com/a/1190000022886063](https://segmentfault.com/a/1190000022886063)