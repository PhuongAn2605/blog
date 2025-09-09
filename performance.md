# Performance

Context: Bảng dữ liệu lớn (≥20k dòng) dùng AG Grid với các tính năng:
- Chỉnh sửa trực tiếp trong AGGrid
- undo/redo
- Show toàn bộ dòng dữ liệu
- Lưu lại từng action của người dùng
- Update lại dữ liệu của 1 row khi row đó bị outdated (row được update bởi người khác nhưng chưa lấy về được dữ liệu mới nhất)
- Disable updating row

Mục tiêu là giữ UI mượt, phản hồi nhanh khi edit/scroll, edit inline, và thao tác phím tắt Ctrl Z / Cmd Z, Ctrl Y / Cmd Y.

## Các giải pháp tối ưu:

- **Virtualization**: Hiển thị toàn bộ dữ liệu, dùng row virtualization mặc định của AG Grid (chỉ render vùng nhìn thấy + buffer)

      <AgGridReact
        rowData={rows}
        columnDefs={columnDefs}
        // Tối ưu cuộn
        rowBuffer={20}
        // Tránh re-render không cần thiết
        getRowId={(p) => p.data.id}
      />

- **Custom cell trong table**: Trước đó dùng CellRenderer nhưng không tốt cho performance nên chuyển sang dùng native component của AGGrid như `cellEditor: "agCheckboxCellEditor"` hoặc `cellEditor: "agSelectCellEditor"`.
- **Custom component**: Với những component bắt buộc phải dùng custom, sử dụng `React.memo` và kiểm tra props bằng hàm so sánh thủ công để hạn chế re-render không cần thiết. Ví dụ:

  ```js
  const MemoizedComponent = memo(SomeComponent, arePropsEqual?)
  ```

- **Lấy state trong store one by one**: Thay vì lấy cả redux state ra, chỉ lấy từng phần cần thiết để hạn chế re-render không cần thiết.

  - Thay vì:
    ```js
    const { undoStack } = useSelector((state: RootState) => state.term);
    ```
    Nên dùng:
    ```js
    const undoStack = useSelector((state: RootState) => state.term.undoStack);
    ```

- **Dùng ImperativeHandle**: Để trigger event từ parent đến child mà không gây ra re-render.
- **Xử lý key press bằng ImperativeHandle**: Thay vì xử lý trong useEffect ở parent component, nên xử lý trực tiếp qua ImperativeHandler để tối ưu hiệu năng.
- **Update data**: Update rowData bằng native function của AGGrid: setDataValue, setData, applyTransaction cho 1 dòng dữ liệu thay vì update data cả table
- **Disable row**: Thêm 1 prop rowDisabled: true/false vào từng row để disable chỉ 1 row khi đang call API update ( update bằng native function của AGGrid)
- Áp dụng React Best Practices cho Aggrid: https://www.ag-grid.com/react-data-grid/react-hooks/ \
      + Use useState/useMemo cho rowData, columnDefs, object properties (defaultColDef, sideBar, statusBar)\
      + Immutable data: use getRowId khi cần update row in table\
      + Use useCallback() cho gridOptions, không bắt buộc dùng useCallback() cho event listeners\
      + Components: sử dụng cellRenderer cần dùng React.memo cho component đó\


<img width="1842" height="926" alt="image" src="https://github.com/user-attachments/assets/9b47957f-0b2b-451d-bccd-58582b32e0de" />


## Tổng kết

Kết hợp các giải pháp trên giúp tối ưu hiệu năng khi làm việc với bảng dữ liệu lớn, đảm bảo trải nghiệm người dùng mượt mà và đáp ứng yêu cầu nghiệp vụ.
