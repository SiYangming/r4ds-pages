# Workflow: pipes {#workflow-pipes}

-   Indenting and line breaks

    
    ```r
    df %>% mutate(y = x + 1)
    # vs
    df %>%
      mutate(
        y = x + 1
      )
    ```

-   `mutate(df, y = x + 1)` vs `df %>% mutate(df, y = x + 1)`

-   with ggplot2

    
    ```r
    df %>% 
      ggplot(aes()) 
    ```

    Don't forget to switch to plus!

-   How long should your pipes be?
    Too long vs too short

-   Restyling with style.
