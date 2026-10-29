
# Ex No: 1 A : To create a employee details fields and to display the employee details using Firebase Database in Android Studio.

## NAME : AADHITHYAA L
## REGISTER NUMBER : 212224220003


## AIM:

To create and display the employee details using Firebase Database in Android Studio.

## EQUIPMENTS REQUIRED:

Android Studio(Min.required Artic Fox)

## ALGORITHM:

Step 1: Open Android Stdio and then click on File -> New -> New project.

Step 2: Then type the Application name as HelloWorld and click Next. 

Step 3: Then select the Minimum SDK as shown below and click Next.

Step 4: Then select the Empty Activity and click Next. Finally click Finish.

Step 5: Design layout in activity_main.xml.

Step 6: Display the employee details in MainActivity file.

Step 7: Save and run the application.

## PROGRAM:
```
Program to print the DatabaseTable using the firebasedatabase”.
Developed by: AADHITHYAA L
Registeration Number : 212224220003
```
## Emp.java : 
```java

package com.example.exp1_pmd;

import androidx.annotation.Keep;

@Keep
@SuppressWarnings("unused")
public class Emp {
    public String id, name, age, salary;

    public Emp() {} // Required for Firebase reflection

    public Emp(String name, String age, String salary) {
        this.name = name;
        this.age = age;
        this.salary = salary;
    }

    public Emp(String id, String name, String age, String salary) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.salary = salary;
    }
}

```

## MainActivity.java : 
```java
package com.example.exp1_pmd;

import android.os.Bundle;
import android.widget.*;
import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import com.google.firebase.database.*;
import java.util.*;

public class MainActivity extends AppCompatActivity {

    private static final String DB_URL = "https://first-exp-459f3-default-rtdb.firebaseio.com/";

    EditText etName, etAge, etSalary;
    Button btnAdd, btnUpdate, btnDelete;
    ListView listView;
    DatabaseReference dbRef;

    ArrayList<String> items = new ArrayList<>();
    ArrayList<String> keys = new ArrayList<>();
    ArrayList<Emp> empList = new ArrayList<>();
    ArrayAdapter<String> adapter;
    String selectedKey = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        etName = findViewById(R.id.etName);
        etAge = findViewById(R.id.etAge);
        etSalary = findViewById(R.id.etSalary);

        btnAdd = findViewById(R.id.btnAdd);
        btnUpdate = findViewById(R.id.btnUpdate);
        btnDelete = findViewById(R.id.btnDelete);
        listView = findViewById(R.id.listView);

        dbRef = FirebaseDatabase.getInstance(DB_URL)
                .getReference("Employees");

        adapter = new ArrayAdapter<>(
                this,
                android.R.layout.simple_list_item_1,
                items
        );

        listView.setAdapter(adapter);

        dbRef.addValueEventListener(new ValueEventListener() {
            @Override
            public void onDataChange(@NonNull DataSnapshot snapshot) {
                items.clear();
                keys.clear();
                empList.clear();

                for (DataSnapshot d : snapshot.getChildren()) {
                    Emp e = d.getValue(Emp.class);

                    if (e != null) {
                        e.id = d.getKey();
                        String nameStr = e.name != null ? e.name : "";
                        String ageStr = e.age != null ? e.age : "";
                        String salaryStr = e.salary != null ? e.salary : "";

                        items.add("Name: " + nameStr + " | Age: " + ageStr + " | Salary: " + salaryStr);
                        keys.add(d.getKey());
                        empList.add(e);
                    }
                }

                adapter.notifyDataSetChanged();
            }

            @Override
            public void onCancelled(@NonNull DatabaseError error) {
                Toast.makeText(MainActivity.this, "Firebase Error: " + error.getMessage(), Toast.LENGTH_LONG).show();
            }
        });

        btnAdd.setOnClickListener(v -> {
            String name = etName.getText().toString().trim();
            String age = etAge.getText().toString().trim();
            String salary = etSalary.getText().toString().trim();

            if (name.isEmpty() || age.isEmpty() || salary.isEmpty()) {
                Toast.makeText(this, "Please fill in all fields", Toast.LENGTH_SHORT).show();
                return;
            }

            Emp e = new Emp(name, age, salary);
            dbRef.push().setValue(e).addOnCompleteListener(task -> {
                if (task.isSuccessful()) {
                    clearFields();
                    Toast.makeText(this, "Employee Added", Toast.LENGTH_SHORT).show();
                } else {
                    String msg = task.getException() != null ? task.getException().getMessage() : "Failed to add";
                    Toast.makeText(this, "Add Failed: " + msg, Toast.LENGTH_LONG).show();
                }
            });
        });

        btnUpdate.setOnClickListener(v -> {
            if (selectedKey != null) {
                String name = etName.getText().toString().trim();
                String age = etAge.getText().toString().trim();
                String salary = etSalary.getText().toString().trim();

                if (name.isEmpty() || age.isEmpty() || salary.isEmpty()) {
                    Toast.makeText(this, "Please fill in all fields", Toast.LENGTH_SHORT).show();
                    return;
                }

                Emp e = new Emp(name, age, salary);
                dbRef.child(selectedKey).setValue(e).addOnCompleteListener(task -> {
                    if (task.isSuccessful()) {
                        clearFields();
                        Toast.makeText(this, "Employee Updated", Toast.LENGTH_SHORT).show();
                    } else {
                        String msg = task.getException() != null ? task.getException().getMessage() : "Failed to update";
                        Toast.makeText(this, "Update Failed: " + msg, Toast.LENGTH_LONG).show();
                    }
                });
            } else {
                Toast.makeText(
                        this,
                        "Select an item first",
                        Toast.LENGTH_SHORT
                ).show();
            }
        });

        btnDelete.setOnClickListener(v -> {
            if (selectedKey != null) {
                dbRef.child(selectedKey).removeValue().addOnCompleteListener(task -> {
                    if (task.isSuccessful()) {
                        clearFields();
                        Toast.makeText(this, "Employee Deleted", Toast.LENGTH_SHORT).show();
                    } else {
                        String msg = task.getException() != null ? task.getException().getMessage() : "Failed to delete";
                        Toast.makeText(this, "Delete Failed: " + msg, Toast.LENGTH_LONG).show();
                    }
                });
            } else {
                Toast.makeText(
                        this,
                        "Select an item first",
                        Toast.LENGTH_SHORT
                ).show();
            }
        });

        listView.setOnItemClickListener((p, v, pos, id) -> {
            if (pos >= 0 && pos < keys.size()) {
                selectedKey = keys.get(pos);
                Emp e = empList.get(pos);

                if (e != null) {
                    etName.setText(e.name != null ? e.name : "");
                    etAge.setText(e.age != null ? e.age : "");
                    etSalary.setText(e.salary != null ? e.salary : "");
                }
            }
        });
    }

    private void clearFields() {
        etName.setText("");
        etAge.setText("");
        etSalary.setText("");
        selectedKey = null;
    }
}

```
## activity_main.xml :
```xml

<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp"
    tools:context=".MainActivity">

    <EditText
        android:id="@+id/etName"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:minHeight="48dp"
        android:autofillHints="name"
        android:hint="@string/hint_name"
        android:inputType="textPersonName"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <EditText
        android:id="@+id/etAge"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:minHeight="48dp"
        android:importantForAutofill="no"
        android:hint="@string/hint_age"
        android:inputType="number"
        android:layout_marginTop="8dp"
        app:layout_constraintTop_toBottomOf="@id/etName"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <EditText
        android:id="@+id/etSalary"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:minHeight="48dp"
        android:importantForAutofill="no"
        android:hint="@string/hint_salary"
        android:inputType="numberDecimal"
        android:layout_marginTop="8dp"
        app:layout_constraintTop_toBottomOf="@id/etAge"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <Button
        android:id="@+id/btnAdd"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:minHeight="48dp"
        android:text="@string/btn_add"
        android:layout_marginTop="16dp"
        app:layout_constraintTop_toBottomOf="@id/etSalary"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toStartOf="@id/btnUpdate"
        app:layout_constraintHorizontal_chainStyle="spread"
        app:layout_constraintHorizontal_weight="1" />

    <Button
        android:id="@+id/btnUpdate"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:minHeight="48dp"
        android:text="@string/btn_update"
        android:layout_marginTop="16dp"
        android:layout_marginStart="8dp"
        app:layout_constraintTop_toBottomOf="@id/etSalary"
        app:layout_constraintStart_toEndOf="@id/btnAdd"
        app:layout_constraintEnd_toStartOf="@id/btnDelete"
        app:layout_constraintHorizontal_weight="1" />

    <Button
        android:id="@+id/btnDelete"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:minHeight="48dp"
        android:text="@string/btn_delete"
        android:layout_marginTop="16dp"
        android:layout_marginStart="8dp"
        app:layout_constraintTop_toBottomOf="@id/etSalary"
        app:layout_constraintStart_toEndOf="@id/btnUpdate"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_weight="1" />

    <ListView
        android:id="@+id/listView"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:layout_marginTop="16dp"
        app:layout_constraintTop_toBottomOf="@id/btnAdd"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

## OUTPUT

<img width="1915" height="1198" alt="Screenshot 2026-10-29 102542" src="https://github.com/user-attachments/assets/92147c2a-ca8f-4028-a60d-5202adc5d449" />
<img width="1901" height="1072" alt="Screenshot 2026-10-29 102518" src="https://github.com/user-attachments/assets/1476698b-2eeb-4664-96bd-4e4e3427ec28" />




## RESULT
Thus a Simple Android Application create a firebase database and to display the employee details using Firbase Real Time Database in Android Studio is developed and executed successfully.
