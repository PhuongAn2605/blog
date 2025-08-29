# Performance

Context: Bảng dữ liệu lớn (≥20k dòng) dùng AG Grid với các tính năng:
- chỉnh sửa trực tiếp trong AGGrid
- undo/redo
- show toàn bộ dòng dữ liệu
- Lưu lại từng action của người dùng
- Update lại dữ liệu của 1 row khi row đó bị outdated (row được update bởi người khác nhưng chưa lấy về được dữ liệu mới nhất)
Mục tiêu là giữ UI mượt, phản hồi nhanh khi edit/scroll, edit inline, và thao tác phím tắt Ctrl Z / Cmd Z, Ctrl Y / Cmd Y.

## Các giải pháp tối ưu:

- **Virtualization**: Hiển thị toàn bộ dữ liệu, dùng row virtualization mặc định của AG Grid (chỉ render vùng nhìn thấy + buffer)

      <AgGridReact
        rowData={rows}
        columnDefs={columnDefs}
        // Tối ưu cuộn
        suppressColumnVirtualisation={false}
        rowBuffer={Math.min(20, Math.ceil(window.innerHeight / 40))}
        // Tránh re-render không cần thiết
        immutableData={true}
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
- **Uơdate data**: Update rowData bằng native function của AGGrid: setDataValue, setData, applyTransaction cho 1 dòng dữ liệu thay vì update data cả table

## Tổng kết

Kết hợp các giải pháp trên giúp tối ưu hiệu năng khi làm việc với bảng dữ liệu lớn, đảm bảo trải nghiệm người dùng mượt mà và đáp ứng yêu cầu nghiệp vụ.
