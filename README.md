# How to control text selection in editable cell in Flutter DataTable (SfDataGrid)?

In this article, we will show you how to control text selection in editable cell in [Flutter DataTable](https://www.syncfusion.com/flutter-widgets/flutter-datagrid).

Initialize the [SfDataGrid](https://pub.dev/documentation/syncfusion_flutter_datagrid/latest/datagrid/SfDataGrid-class.html) widget with all the necessary properties. Using a [ValueNotifier](https://api.flutter.dev/flutter/foundation/ValueNotifier-class.html), we can track the double-tap state, which helps differentiate between a custom edit mode activation and a double-click activation. A FocusNode is used inside the TextField to manage focus. When the field gains focus, the flag determines whether all text should be selected upon entering edit mode via a single click or custom editing (using the Space key) while preventing text selection on a double-click.

By implementing this approach—setting `EditingGestureType.tap` or triggering editing using the Space key—the text gets selected. However, when using `EditingGestureType.doubleTap` and performing a double tap, the cell enters edit mode without selecting the text.

```dart
class EmployeeDataSource extends DataGridSource {
  /// Creates the employee data source class with required details.
  EmployeeDataSource(
      {required this.employees, required this.isDoubleTapNotifier}) {
    buildDataGridSource(employees);
  }

  void buildDataGridSource(List<Employee> employees) {
    _employeeData = employees
        .map<DataGridRow>((e) => DataGridRow(cells: [
              DataGridCell<int>(columnName: 'ID', value: e.id),
              DataGridCell<String>(columnName: 'Name', value: e.name),
              DataGridCell<String>(
                  columnName: 'Designation', value: e.designation),
              DataGridCell<int>(columnName: 'Salary', value: e.salary),
              DataGridCell<String>(columnName: 'Country', value: e.country),
            ]))
        .toList();
  }

  ValueNotifier<bool> isDoubleTapNotifier;

 TextEditingController editingController = TextEditingController();

  @override
  Widget? buildEditWidget(DataGridRow dataGridRow,
      RowColumnIndex rowColumnIndex, GridColumn column, CellSubmit submitCell) {
    final String displayText = dataGridRow
            .getCells()
            .firstWhereOrNull((DataGridCell dataGridCell) =>
                dataGridCell.columnName == column.columnName)
            ?.value
            ?.toString() ??
        '';

    newCellValue = null;

    final bool isNumericType =
        column.columnName == 'ID' || column.columnName == 'Salary';

    final FocusNode focusNode = FocusNode();
    // Select all text on edit mode, but disable selection on double-click.
    focusNode.addListener(() {
      if (focusNode.hasFocus && !isDoubleTapNotifier.value) {
        editingController.selection = TextSelection(
          baseOffset: 0,
          extentOffset: editingController.text.length,
        );
      }
    });

    return Container(
      padding: const EdgeInsets.all(8.0),
      alignment: isNumericType ? Alignment.centerRight : Alignment.centerLeft,
      child: TextField(
        autofocus: true,
        focusNode: focusNode,
        controller: editingController..text = displayText,
        textAlign: isNumericType ? TextAlign.right : TextAlign.left,
        decoration: const InputDecoration(
          contentPadding: EdgeInsets.fromLTRB(0, 0, 0, 8.0),
        ),
        keyboardType: isNumericType ? TextInputType.number : TextInputType.text,
        onChanged: (String value) {
          if (value.isNotEmpty) {
            if (isNumericType) {
              newCellValue = int.parse(value);
            } else {
              newCellValue = value;
            }
          } else {
            newCellValue = null;
          }
        },
        onSubmitted: (String value) {
          submitCell();
        },
      ),
    );
  }
}

class CustomSelectionManager extends RowSelectionManager {
  CustomSelectionManager(
      this.dataGridController, this.context, this.isDoubleTapNotifier);
  DataGridController dataGridController;
  BuildContext context;
  ValueNotifier<bool> isDoubleTapNotifier;

  @override
  Future<void> handleKeyEvent(KeyEvent keyEvent) async {
    // Handle space key press to initiate editing mode.
    if (keyEvent.logicalKey == LogicalKeyboardKey.space) {
      isDoubleTapNotifier.value = false;
      dataGridController.beginEdit(dataGridController.currentCell);
    } else {
      super.handleKeyEvent(keyEvent);
    }
  }
}
```

You can download this example on [GitHub](https://github.com/SyncfusionExamples/How-to-control-text-selection-in-editable-cell-in-Flutter-DataTable).