package org.example.Service;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.example.Exception.EmployeeAlreadyExistsException;
import org.example.Exception.NoSuchEmployeeExistsException;
import org.example.model.Employee;
import org.example.repository.EmployeeRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class EmployeeServiceImpl implements EmployeeService {

    private static final Logger log = LogManager.getLogger(EmployeeServiceImpl.class);

    @Autowired
    private EmployeeRepository employeeRepository;

    @Override
    public Employee addEmployee(Employee employee) {
        log.info("Adding employee: " + employee);
        if (employeeRepository.findByMobileNo(employee.getMobileNo()).isPresent()) {
            log.error("Employee with mobile number " + employee.getMobileNo() + " already exists");
            throw new EmployeeAlreadyExistsException("Employee already exists!!");
        }
        return employeeRepository.save(employee);
    }

    @Override
    public void deleteEmployee(Long id) {
        if (!employeeRepository.existsById(id)) {
            throw new NoSuchEmployeeExistsException("Employee with ID " + id + " not found");
        }
        employeeRepository.deleteById(id);
    }

    @Override
    public Employee updateEmployee(Long id, Employee updatedEmployee) {
        Employee existingEmployee = employeeRepository.findById(id)
                .orElseThrow(() -> new NoSuchEmployeeExistsException("Employee not found with ID: " + id));
        existingEmployee.setFirstName(updatedEmployee.getFirstName());
        existingEmployee.setLastName(updatedEmployee.getLastName());
        existingEmployee.setMobileNo(updatedEmployee.getMobileNo());
        existingEmployee.setAge(updatedEmployee.getAge());
        return employeeRepository.save(existingEmployee);
    }

    @Override
    public List<Employee> getAllEmployees() {
        return employeeRepository.findAll();
    }
}
