# Using the Grid Control in a Doc/View framework

A simple tutorial that demonstrates how to use the grid control in a doc/view application.

- [Download full grid demo project - 315.1 KB](https://raw.githubusercontent.com/ChrisMaunder/grid_in_view/master/docs/assets/gridctrl_demo227.zip)

![gridctrl example image](https://raw.githubusercontent.com/ChrisMaunder/grid_in_view/master/docs/assets/grid_in_view.gif) 

I have had many, MANY questions asking how to use my [MFC grid control](http://www.codeproject.com/miscctrl/gridctrl.asp) in a view instead of in a dialog, so hopefully this will help. 

The easiest way as I see it is as follows: 

1. Add a member variable of type `CGridCtrl*` to your view class: 

    ```
    CGridCtrl* m_pGrid;
    ```
2. Initialise this to NULL in your view class' constructor: 

    ```
    CMyView::CMyView
    {
        m_pGrid = NULL;
    }
    ```
3. In the `CView` function `OnInitialUpdate`, create a new `CGridCtrl` object if the `m_pGrid` is not NULL, and then create the `CGridCtrl` window: 

    ```
    CMyView::OnInitialUpdate
    {
        CView::OnInitialUpdate();
    
        if (m_pGrid == NULL)             // Have we already done this bit?
        {
            m_pGrid = new CGridCtrl;     // Create the Gridctrl object
            if (!m_pGrid ) return;
    
            CRect rect;                  // Create the Gridctrl window
            GetClientRect(rect);
            m_pGrid->Create(rect, this, 100);
    
            m_pGrid->SetRowCount(50);     // fill it up with stuff
            m_pGrid->SetColumnCount(10);
            
            // ... etc
        }
    }
    ```

    This allows the view to be reused (eg SDI situations).
4. We want the grid to take up the whole of the view's client space, so add a handler to the `WM_SIZE` message for the view and edit the `OnSize` function thus: 

    ```
    CMyView::OnSize(UINT nType, int cx, int cy) 
    {
        CView::OnSize(nType, cx, cy);
        
        if (m_pGrid->GetSafeHwnd())     // Have the grid object and window 
        {                               // been created yet?
            CRect rect;
            GetClientRect(rect);        // Get the size of the view's client
                                        // area
            m_pGrid->MoveWindow(rect);  // Resize the grid to take up that 
                                        // space.
        }
    }
    ```
5. Remember to delete the object when you are done: 

    ```
    CMyView::~CMyView
    {
        delete m_pGrid;
    }
    ```
6. You may want to also add an `OnCmdMsg` overide to your view class and let the grid control have first go at the messages (this will allow commands such as `ID_EDIT_COPY` to be wired in automatically: 

    ```
    BOOL CMyView::OnCmdMsg(UINT nID, int nCode, void* pExtra, 
                           AFX_CMDHANDLERINFO* pHandlerInfo) 
    {
        if (m_pGrid && IsWindow(m_pGrid->m_hWnd))
            if (m_pGrid->OnCmdMsg(nID, nCode, pExtra, pHandlerInfo))
                return TRUE;
    
        return CView::OnCmdMsg(nID, nCode, pExtra, pHandlerInfo);
    }
    ```

If you want print preview, then check out Koay Kah Hoe's article [Print Previewing without the Document/View Framework](http://www.codeproject.com/printing/print_preview.asp).
