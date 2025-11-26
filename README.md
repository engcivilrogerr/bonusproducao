import React, { useState, useEffect } from 'react';
import { Plus, Edit2, Trash2, Users, Calendar, DollarSign, FileText, Download, Printer, Upload, Save } from 'lucide-react';

const BonusManagerApp = () => {
  const [activeTab, setActiveTab] = useState('dashboard');
  const [employees, setEmployees] = useState([]);
  const [productions, setProductions] = useState([]);
  const [selectedTeam, setSelectedTeam] = useState('Equipe 1');
  const [showEmployeeForm, setShowEmployeeForm] = useState(false);
  const [editingEmployee, setEditingEmployee] = useState(null);
  const [startDate, setStartDate] = useState('2025-10-21');
  const [endDate, setEndDate] = useState('2025-11-09');
  const [showDateConfig, setShowDateConfig] = useState(false);
  const [selectedProductionCell, setSelectedProductionCell] = useState(null);
  const [showProductionModal, setShowProductionModal] = useState(false);

  const teams = ['Equipe 1', 'Equipe 2', 'Equipe 3', 'Equipe 4', 'Equipe 5', 'Equipe 6', 'Apoio'];
  const positions = [
    { name: 'Encarregado', baseValue: 60 },
    { name: 'Enc de Obras', baseValue: 60 },
    { name: 'Engenheiro', baseValue: 80 },
    { name: 'Operador de Retro', baseValue: 50 },
    { name: 'Op. Maq. Pesadas', baseValue: 50 },
    { name: 'Bombeiro', baseValue: 50 },
    { name: 'Pedreiro', baseValue: 50 },
    { name: 'Servente', baseValue: 50 },
    { name: 'Mot. Caminhão', baseValue: 40 }
  ];

  const addEmployee = (employee) => {
    if (editingEmployee) {
      setEmployees(employees.map(e => e.id === editingEmployee.id ? { ...employee, id: editingEmployee.id } : e));
      setEditingEmployee(null);
    } else {
      setEmployees([...employees, { ...employee, id: Date.now() }]);
    }
    setShowEmployeeForm(false);
  };

  const deleteEmployee = (id) => {
    if (window.confirm('Deseja realmente excluir este funcionário?')) {
      setEmployees(employees.filter(e => e.id !== id));
    }
  };

  const updateProduction = (date, employeeId, value, workedTeam = null, observation = '') => {
    const key = `${date}-${employeeId}`;
    const existing = productions.find(p => p.key === key);
    
    if (existing) {
      setProductions(productions.map(p => p.key === key ? { 
        ...p, 
        value: parseFloat(value) || 0,
        workedTeam: workedTeam || p.workedTeam,
        observation: observation !== undefined ? observation : p.observation
      } : p));
    } else {
      setProductions([...productions, { 
        key, 
        date, 
        employeeId, 
        value: parseFloat(value) || 0,
        workedTeam,
        observation
      }]);
    }
  };

  const getProductionValue = (date, employeeId) => {
    const prod = productions.find(p => p.date === date && p.employeeId === employeeId);
    return prod ? prod.value : 0;
  };

  const getProductionDetails = (date, employeeId) => {
    return productions.find(p => p.date === date && p.employeeId === employeeId);
  };

  const openProductionModal = (date, employee) => {
    const details = getProductionDetails(date, employee.id);
    setSelectedProductionCell({
      date,
      employee,
      value: details?.value || 0,
      workedTeam: details?.workedTeam || null,
      observation: details?.observation || ''
    });
    setShowProductionModal(true);
  };

  const saveProductionDetails = () => {
    if (selectedProductionCell) {
      updateProduction(
        selectedProductionCell.date,
        selectedProductionCell.employee.id,
        selectedProductionCell.value,
        selectedProductionCell.workedTeam,
        selectedProductionCell.observation
      );
      setShowProductionModal(false);
      setSelectedProductionCell(null);
    }
  };

  const calculateEmployeeTotal = (employeeId) => {
    return productions
      .filter(p => p.employeeId === employeeId)
      .reduce((sum, p) => sum + p.value, 0);
  };

  const getDatesForMonth = () => {
    const dates = [];
    const start = new Date(startDate);
    const end = new Date(endDate);
    
    for (let d = new Date(start); d <= end; d.setDate(d.getDate() + 1)) {
      const day = String(d.getDate()).padStart(2, '0');
      const month = String(d.getMonth() + 1).padStart(2, '0');
      const year = d.getFullYear();
      dates.push(`${day}/${month}/${year}`);
    }
    
    return dates;
  };

  const teamEmployees = employees.filter(e => e.team === selectedTeam);
  const dates = getDatesForMonth();

  const handlePrint = () => {
    window.print();
  };

  const generatePDF = () => {
    const printContent = document.getElementById('print-area');
    const originalContent = document.body.innerHTML;
    const printButton = document.querySelector('.no-print');
    
    if (printButton) printButton.style.display = 'none';
    
    document.body.innerHTML = printContent.innerHTML;
    window.print();
    document.body.innerHTML = originalContent;
    window.location.reload();
  };

  const exportProductionToPDF = () => {
    let content = `
      <div style="font-family: Arial, sans-serif; padding: 20px;">
        <h1 style="text-align: center; color: #1e40af;">Relatório de Produção - ${selectedTeam}</h1>
        <h3 style="text-align: center; color: #6b7280;">Attalea Construções Ltda</h3>
        <p style="text-align: center; color: #6b7280;">Período: ${dates[0]} a ${dates[dates.length - 1]}</p>
        <hr style="margin: 20px 0;">
        <table style="width: 100%; border-collapse: collapse; margin-top: 20px;">
          <thead>
            <tr style="background-color: #e5e7eb;">
              <th style="border: 1px solid #d1d5db; padding: 8px; text-align: left;">Funcionário</th>
              <th style="border: 1px solid #d1d5db; padding: 8px; text-align: left;">Cargo</th>
              <th style="border: 1px solid #d1d5db; padding: 8px; text-align: right;">Valor Diário</th>
              <th style="border: 1px solid #d1d5db; padding: 8px; text-align: right;">Total</th>
            </tr>
          </thead>
          <tbody>
    `;

    teamEmployees.forEach(emp => {
      const total = calculateEmployeeTotal(emp.id);
      content += `
        <tr>
          <td style="border: 1px solid #d1d5db; padding: 8px;">${emp.name}</td>
          <td style="border: 1px solid #d1d5db; padding: 8px;">${emp.position}</td>
          <td style="border: 1px solid #d1d5db; padding: 8px; text-align: right;">R$ ${emp.dailyValue.toFixed(2)}</td>
          <td style="border: 1px solid #d1d5db; padding: 8px; text-align: right; font-weight: bold; color: #059669;">R$ ${total.toFixed(2)}</td>
        </tr>
      `;
    });

    const teamTotal = teamEmployees.reduce((sum, emp) => sum + calculateEmployeeTotal(emp.id), 0);
    content += `
          </tbody>
          <tfoot>
            <tr style="background-color: #f3f4f6; font-weight: bold;">
              <td colspan="3" style="border: 1px solid #d1d5db; padding: 8px; text-align: right;">TOTAL DA EQUIPE:</td>
              <td style="border: 1px solid #d1d5db; padding: 8px; text-align: right; color: #059669;">R$ ${teamTotal.toFixed(2)}</td>
            </tr>
          </tfoot>
        </table>
      </div>
    `;

    const printWindow = window.open('', '', 'width=800,height=600');
    printWindow.document.write(content);
    printWindow.document.close();
    printWindow.print();
  };

  const exportData = () => {
    const dataToExport = {
      employees,
      productions,
      startDate,
      endDate,
      exportDate: new Date().toISOString(),
      version: '1.0'
    };
    
    const dataStr = JSON.stringify(dataToExport, null, 2);
    const dataBlob = new Blob([dataStr], { type: 'application/json' });
    const url = URL.createObjectURL(dataBlob);
    const link = document.createElement('a');
    link.href = url;
    link.download = `bonus-backup-${new Date().toISOString().split('T')[0]}.json`;
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    URL.revokeObjectURL(url);
    
    alert('✅ Dados exportados com sucesso! Arquivo salvo na pasta de Downloads.');
  };

  const importData = (event) => {
    const file = event.target.files[0];
    if (!file) return;

    const reader = new FileReader();
    reader.onload = (e) => {
      try {
        const importedData = JSON.parse(e.target.result);
        
        if (!importedData.employees || !importedData.productions) {
          alert('❌ Arquivo inválido! Certifique-se de importar um arquivo de backup válido.');
          return;
        }

        const confirmImport = window.confirm(
          `📥 Importar dados?\n\n` +
          `Funcionários: ${importedData.employees.length}\n` +
          `Registros de produção: ${importedData.productions.length}\n` +
          `Data do backup: ${new Date(importedData.exportDate).toLocaleString('pt-BR')}\n\n` +
          `⚠️ ATENÇÃO: Isso irá SUBSTITUIR todos os dados atuais!`
        );

        if (confirmImport) {
          setEmployees(importedData.employees);
          setProductions(importedData.productions);
          if (importedData.startDate) setStartDate(importedData.startDate);
          if (importedData.endDate) setEndDate(importedData.endDate);
          
          alert('✅ Dados importados com sucesso!');
        }
      } catch (error) {
        alert('❌ Erro ao importar arquivo! Verifique se o arquivo está correto.');
        console.error('Import error:', error);
      }
    };
    reader.readAsText(file);
    event.target.value = '';
  };

  const exportReportToPDF = () => {
    let content = `
      <div style="font-family: Arial, sans-serif; padding: 20px;">
        <h1 style="text-align: center; color: #1e40af;">Relatório Completo de Bônus</h1>
        <h3 style="text-align: center; color: #6b7280;">Attalea Construções Ltda - Rede de Esgoto</h3>
        <p style="text-align: center; color: #6b7280;">Período: ${dates[0]} a ${dates[dates.length - 1]}</p>
        <hr style="margin: 20px 0;">
    `;

    teams.forEach(team => {
      const teamEmps = employees.filter(e => e.team === team);
      if (teamEmps.length === 0) return;
      
      const teamTotal = teamEmps.reduce((sum, emp) => sum + calculateEmployeeTotal(emp.id), 0);
      
      content += `
        <div style="margin-bottom: 30px; page-break-inside: avoid;">
          <h2 style="background-color: #dbeafe; padding: 10px; margin-bottom: 10px;">${team} - Total: R$ ${teamTotal.toFixed(2)}</h2>
          <table style="width: 100%; border-collapse: collapse; margin-bottom: 20px;">
            <thead>
              <tr style="background-color: #e5e7eb;">
                <th style="border: 1px solid #d1d5db; padding: 8px; text-align: left;">Funcionário</th>
                <th style="border: 1px solid #d1d5db; padding: 8px; text-align: left;">Cargo</th>
                <th style="border: 1px solid #d1d5db; padding: 8px; text-align: center;">Ficha</th>
                <th style="border: 1px solid #d1d5db; padding: 8px; text-align: right;">Valor/Dia</th>
                <th style="border: 1px solid #d1d5db; padding: 8px; text-align: center;">Dias</th>
                <th style="border: 1px solid #d1d5db; padding: 8px; text-align: right;">Total</th>
              </tr>
            </thead>
            <tbody>
      `;

      teamEmps.sort((a, b) => calculateEmployeeTotal(b.id) - calculateEmployeeTotal(a.id)).forEach(emp => {
        const total = calculateEmployeeTotal(emp.id);
        const days = productions.filter(p => p.employeeId === emp.id && p.value > 0).length;
        const workedOtherTeams = productions.filter(p => p.employeeId === emp.id && p.workedTeam).length;
        const hasObservations = productions.some(p => p.employeeId === emp.id && p.observation);
        
        content += `
          <tr>
            <td style="border: 1px solid #d1d5db; padding: 8px;">
              ${emp.name}
              ${workedOtherTeams > 0 ? '<br><span style="color: #ea580c; font-size: 11px;">⚠️ Trabalhou em outras equipes</span>' : ''}
            </td>
            <td style="border: 1px solid #d1d5db; padding: 8px;">${emp.position}</td>
            <td style="border: 1px solid #d1d5db; padding: 8px; text-align: center;">${emp.badge || '-'}</td>
            <td style="border: 1px solid #d1d5db; padding: 8px; text-align: right;">R$ ${emp.dailyValue.toFixed(2)}</td>
            <td style="border: 1px solid #d1d5db; padding: 8px; text-align: center;">${days}</td>
            <td style="border: 1px solid #d1d5db; padding: 8px; text-align: right; font-weight: bold; color: #059669;">R$ ${total.toFixed(2)}</td>
          </tr>
        `;
        
        // Adicionar detalhes de outras equipes e observações
        const otherTeamDetails = productions.filter(p => p.employeeId === emp.id && (p.workedTeam || p.observation));
        if (otherTeamDetails.length > 0) {
          content += `
            <tr>
              <td colspan="6" style="border: 1px solid #d1d5db; padding: 8px; background-color: #fef3c7; font-size: 11px;">
                <strong>Detalhes:</strong><br>
          `;
          otherTeamDetails.forEach(detail => {
            if (detail.workedTeam || detail.observation) {
              content += `• ${detail.date}: `;
              if (detail.workedTeam) content += `Trabalhou na ${detail.workedTeam}. `;
              if (detail.observation) content += `Obs: ${detail.observation}`;
              content += `<br>`;
            }
          });
          content += `
              </td>
            </tr>
          `;
        }
      });

      content += `
            </tbody>
          </table>
        </div>
      `;
    });

    const grandTotal = employees.reduce((sum, emp) => sum + calculateEmployeeTotal(emp.id), 0);
    content += `
        <div style="margin-top: 30px; padding: 15px; background-color: #f3f4f6; border: 2px solid #1e40af;">
          <h2 style="text-align: center; color: #1e40af;">TOTAL GERAL: R$ ${grandTotal.toFixed(2)}</h2>
          <p style="text-align: center; color: #6b7280;">${employees.length} funcionários em ${teams.filter(t => employees.some(e => e.team === t)).length} equipes</p>
        </div>
      </div>
    `;

    const printWindow = window.open('', '', 'width=800,height=600');
    printWindow.document.write(content);
    printWindow.document.close();
    printWindow.print();
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 p-4">
      <div className="max-w-7xl mx-auto">
        <div className="bg-white rounded-lg shadow-lg p-6 mb-6">
          <h1 className="text-3xl font-bold text-gray-800 mb-2">
            Sistema de Gestão de Bônus de Produção
          </h1>
          <p className="text-gray-600 mb-4">Attalea Construções Ltda - Rede de Esgoto</p>
          
          <div className="flex flex-wrap gap-2">
            <button
              onClick={exportData}
              className="flex items-center gap-2 bg-green-600 text-white px-4 py-2 rounded-lg hover:bg-green-700 transition-colors text-sm"
            >
              <Download size={18} />
              Exportar Dados
            </button>
            
            <label className="flex items-center gap-2 bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 transition-colors cursor-pointer text-sm">
              <Upload size={18} />
              Importar Dados
              <input
                type="file"
                accept=".json"
                onChange={importData}
                className="hidden"
              />
            </label>
            
            <div className="ml-auto text-right text-sm text-gray-500">
              <p>💾 Dados salvos automaticamente</p>
              <p className="text-xs">Último acesso: {new Date().toLocaleString('pt-BR')}</p>
            </div>
          </div>
        </div>

        <div className="bg-white rounded-lg shadow-lg mb-6">
          <div className="flex border-b">
            <button
              onClick={() => setActiveTab('dashboard')}
              className={`flex items-center gap-2 px-6 py-4 font-medium transition-colors ${
                activeTab === 'dashboard'
                  ? 'border-b-2 border-blue-500 text-blue-600'
                  : 'text-gray-600 hover:text-gray-800'
              }`}
            >
              <FileText size={20} />
              Dashboard
            </button>
            <button
              onClick={() => setActiveTab('employees')}
              className={`flex items-center gap-2 px-6 py-4 font-medium transition-colors ${
                activeTab === 'employees'
                  ? 'border-b-2 border-blue-500 text-blue-600'
                  : 'text-gray-600 hover:text-gray-800'
              }`}
            >
              <Users size={20} />
              Funcionários
            </button>
            <button
              onClick={() => setActiveTab('production')}
              className={`flex items-center gap-2 px-6 py-4 font-medium transition-colors ${
                activeTab === 'production'
                  ? 'border-b-2 border-blue-500 text-blue-600'
                  : 'text-gray-600 hover:text-gray-800'
              }`}
            >
              <Calendar size={20} />
              Produção Diária
            </button>
            <button
              onClick={() => setActiveTab('report')}
              className={`flex items-center gap-2 px-6 py-4 font-medium transition-colors ${
                activeTab === 'report'
                  ? 'border-b-2 border-blue-500 text-blue-600'
                  : 'text-gray-600 hover:text-gray-800'
              }`}
            >
              <DollarSign size={20} />
              Relatório Completo
            </button>
          </div>
        </div>

        {activeTab === 'dashboard' && (
          <div className="space-y-6">
            <div className="bg-blue-50 border-l-4 border-blue-500 p-4 mb-4">
              <div className="flex items-start">
                <Save className="text-blue-500 mr-3 flex-shrink-0" size={24} />
                <div>
                  <h3 className="font-bold text-blue-800 mb-1">💡 Dica: Faça Backup Regular!</h3>
                  <p className="text-sm text-blue-700">
                    Use o botão "Exportar Dados" acima para salvar um arquivo de backup. 
                    Assim você pode transferir seus dados entre celular e computador, ou recuperá-los se limpar o navegador.
                  </p>
                </div>
              </div>
            </div>

            <div className="flex justify-end mb-4 no-print">
              <button
                onClick={handlePrint}
                className="flex items-center gap-2 bg-gray-600 text-white px-4 py-2 rounded-lg hover:bg-gray-700 transition-colors"
              >
                <Printer size={20} />
                Imprimir Dashboard
              </button>
            </div>
            
            <div id="print-area">
              <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
              <div className="bg-white rounded-lg shadow-lg p-6">
                <div className="flex items-center gap-4">
                  <div className="bg-blue-100 p-3 rounded-lg">
                    <Users className="text-blue-600" size={24} />
                  </div>
                  <div>
                    <p className="text-gray-600 text-sm">Total de Funcionários</p>
                    <p className="text-2xl font-bold text-gray-800">{employees.length}</p>
                  </div>
                </div>
              </div>
              
              <div className="bg-white rounded-lg shadow-lg p-6">
                <div className="flex items-center gap-4">
                  <div className="bg-green-100 p-3 rounded-lg">
                    <DollarSign className="text-green-600" size={24} />
                  </div>
                  <div>
                    <p className="text-gray-600 text-sm">Bônus Total (Mês)</p>
                    <p className="text-2xl font-bold text-gray-800">
                      R$ {employees.reduce((sum, emp) => sum + calculateEmployeeTotal(emp.id), 0).toFixed(2)}
                    </p>
                  </div>
                </div>
              </div>
              
              <div className="bg-white rounded-lg shadow-lg p-6">
                <div className="flex items-center gap-4">
                  <div className="bg-purple-100 p-3 rounded-lg">
                    <Calendar className="text-purple-600" size={24} />
                  </div>
                  <div>
                    <p className="text-gray-600 text-sm">Equipes Ativas</p>
                    <p className="text-2xl font-bold text-gray-800">{teams.length}</p>
                  </div>
                </div>
              </div>
            </div>

            <div className="bg-white rounded-lg shadow-lg p-6">
              <h2 className="text-xl font-bold text-gray-800 mb-4">Resumo por Equipe</h2>
              <div className="space-y-3">
                {teams.map(team => {
                  const teamEmps = employees.filter(e => e.team === team);
                  const teamTotal = teamEmps.reduce((sum, emp) => sum + calculateEmployeeTotal(emp.id), 0);
                  return (
                    <div key={team} className="flex justify-between items-center p-4 bg-gray-50 rounded-lg">
                      <div>
                        <p className="font-semibold text-gray-800">{team}</p>
                        <p className="text-sm text-gray-600">{teamEmps.length} funcionários</p>
                      </div>
                      <p className="text-lg font-bold text-green-600">R$ {teamTotal.toFixed(2)}</p>
                    </div>
                  );
                })}
              </div>
            </div>
            </div>
          </div>
        )}

        {activeTab === 'employees' && (
          <div className="bg-white rounded-lg shadow-lg p-6">
            <div className="flex justify-between items-center mb-6">
              <h2 className="text-xl font-bold text-gray-800">Gerenciar Funcionários</h2>
              <button
                onClick={() => {
                  setEditingEmployee(null);
                  setShowEmployeeForm(true);
                }}
                className="flex items-center gap-2 bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 transition-colors"
              >
                <Plus size={20} />
                Adicionar Funcionário
              </button>
            </div>

            {showEmployeeForm && (
              <EmployeeForm
                positions={positions}
                teams={teams}
                onSave={addEmployee}
                onCancel={() => {
                  setShowEmployeeForm(false);
                  setEditingEmployee(null);
                }}
                initialData={editingEmployee}
              />
            )}

            <div className="space-y-4 mt-6">
              {teams.map(team => {
                const teamEmps = employees.filter(e => e.team === team);
                if (teamEmps.length === 0) return null;
                
                return (
                  <div key={team} className="border rounded-lg p-4">
                    <h3 className="font-bold text-gray-800 mb-3">{team}</h3>
                    <div className="space-y-2">
                      {teamEmps.map(emp => (
                        <div key={emp.id} className="flex justify-between items-center p-3 bg-gray-50 rounded">
                          <div>
                            <p className="font-semibold text-gray-800">{emp.name}</p>
                            <p className="text-sm text-gray-600">
                              {emp.position} - R$ {emp.dailyValue.toFixed(2)}/dia
                              {emp.badge && <span className="ml-2 text-xs bg-blue-100 text-blue-600 px-2 py-1 rounded">
                                Ficha: {emp.badge}
                              </span>}
                            </p>
                          </div>
                          <div className="flex gap-2">
                            <button
                              onClick={() => {
                                setEditingEmployee(emp);
                                setShowEmployeeForm(true);
                              }}
                              className="p-2 text-blue-600 hover:bg-blue-50 rounded"
                            >
                              <Edit2 size={18} />
                            </button>
                            <button
                              onClick={() => deleteEmployee(emp.id)}
                              className="p-2 text-red-600 hover:bg-red-50 rounded"
                            >
                              <Trash2 size={18} />
                            </button>
                          </div>
                        </div>
                      ))}
                    </div>
                  </div>
                );
              })}
            </div>
          </div>
        )}

        {activeTab === 'production' && (
          <div className="bg-white rounded-lg shadow-lg p-6">
            <div className="flex justify-between items-center mb-6">
              <div className="flex-1">
                <label className="block text-sm font-medium text-gray-700 mb-2">
                  Selecionar Equipe
                </label>
                <select
                  value={selectedTeam}
                  onChange={(e) => setSelectedTeam(e.target.value)}
                  className="w-full md:w-64 px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
                >
                  {teams.map(team => (
                    <option key={team} value={team}>{team}</option>
                  ))}
                </select>
              </div>
              <div className="flex gap-2 no-print">
                <button
                  onClick={() => setShowDateConfig(!showDateConfig)}
                  className="flex items-center gap-2 bg-indigo-600 text-white px-4 py-2 rounded-lg hover:bg-indigo-700 transition-colors"
                >
                  <Calendar size={20} />
                  Configurar Período
                </button>
                <button
                  onClick={exportProductionToPDF}
                  className="flex items-center gap-2 bg-green-600 text-white px-4 py-2 rounded-lg hover:bg-green-700 transition-colors"
                >
                  <Download size={20} />
                  Exportar PDF
                </button>
              </div>
            </div>

            {showDateConfig && (
              <div className="bg-indigo-50 p-4 rounded-lg mb-6">
                <h3 className="font-bold text-gray-800 mb-3">Configurar Período de Produção</h3>
                <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-1">Data Inicial</label>
                    <input
                      type="date"
                      value={startDate}
                      onChange={(e) => setStartDate(e.target.value)}
                      className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500"
                    />
                  </div>
                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-1">Data Final</label>
                    <input
                      type="date"
                      value={endDate}
                      onChange={(e) => setEndDate(e.target.value)}
                      className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500"
                    />
                  </div>
                </div>
                <p className="text-sm text-gray-600 mt-3">
                  Total de dias no período: {dates.length}
                </p>
              </div>
            )}

            {teamEmployees.length === 0 ? (
              <div className="text-center py-12 text-gray-500">
                Nenhum funcionário cadastrado nesta equipe.
              </div>
            ) : (
              <div className="overflow-x-auto">
                <table className="w-full text-sm">
                  <thead>
                    <tr className="bg-gray-100">
                      <th className="px-4 py-2 text-left font-semibold text-gray-700 sticky left-0 bg-gray-100 z-10">
                        Funcionário
                      </th>
                      <th className="px-4 py-2 text-left font-semibold text-gray-700">Cargo</th>
                      {dates.map(date => (
                        <th key={date} className="px-2 py-2 text-center font-semibold text-gray-700 whitespace-nowrap">
                          {date.substring(0, 5)}
                        </th>
                      ))}
                      <th className="px-4 py-2 text-right font-semibold text-gray-700">Total</th>
                    </tr>
                  </thead>
                  <tbody>
                    {teamEmployees.map(emp => (
                      <tr key={emp.id} className="border-b hover:bg-gray-50">
                        <td className="px-4 py-2 font-medium text-gray-800 sticky left-0 bg-white z-10">
                          {emp.name}
                        </td>
                        <td className="px-4 py-2 text-gray-600">{emp.position}</td>
                        {dates.map(date => (
                          <td key={date} className="px-2 py-2">
                            <div className="relative">
                              <input
                                type="number"
                                step="0.01"
                                value={getProductionValue(date, emp.id) || ''}
                                onChange={(e) => updateProduction(date, emp.id, e.target.value)}
                                onClick={() => openProductionModal(date, emp)}
                                className="w-16 px-2 py-1 border border-gray-300 rounded text-center focus:ring-2 focus:ring-blue-500 focus:border-transparent cursor-pointer"
                                placeholder="0"
                                title="Clique para adicionar detalhes"
                              />
                              {getProductionDetails(date, emp.id)?.workedTeam && (
                                <span className="absolute -top-1 -right-1 w-3 h-3 bg-orange-500 rounded-full" title="Trabalhou em outra equipe"></span>
                              )}
                              {getProductionDetails(date, emp.id)?.observation && (
                                <span className="absolute -bottom-1 -right-1 w-3 h-3 bg-blue-500 rounded-full" title="Possui observação"></span>
                              )}
                            </div>
                          </td>
                        ))}
                        <td className="px-4 py-2 text-right font-bold text-green-600">
                          R$ {calculateEmployeeTotal(emp.id).toFixed(2)}
                        </td>
                      </tr>
                    ))}
                  </tbody>
                  <tfoot>
                    <tr className="bg-gray-100 font-bold">
                      <td colSpan={2} className="px-4 py-3 text-gray-800">TOTAL DA EQUIPE</td>
                      {dates.map(date => {
                        const dayTotal = teamEmployees.reduce((sum, emp) => 
                          sum + getProductionValue(date, emp.id), 0
                        );
                        return (
                          <td key={date} className="px-2 py-3 text-center text-gray-800">
                            {dayTotal > 0 ? dayTotal.toFixed(0) : '-'}
                          </td>
                        );
                      })}
                      <td className="px-4 py-3 text-right text-green-600">
                        R$ {teamEmployees.reduce((sum, emp) => sum + calculateEmployeeTotal(emp.id), 0).toFixed(2)}
                      </td>
                    </tr>
                  </tfoot>
                </table>
              </div>
            )}
          </div>
        )}

        {showProductionModal && selectedProductionCell && (
          <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4">
            <div className="bg-white rounded-lg shadow-xl max-w-md w-full p-6">
              <h3 className="text-xl font-bold text-gray-800 mb-4">
                Detalhes da Produção
              </h3>
              
              <div className="space-y-4">
                <div>
                  <p className="text-sm text-gray-600">Funcionário</p>
                  <p className="font-semibold text-gray-800">{selectedProductionCell.employee.name}</p>
                </div>
                
                <div>
                  <p className="text-sm text-gray-600">Data</p>
                  <p className="font-semibold text-gray-800">{selectedProductionCell.date}</p>
                </div>
                
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Valor do Bônus (R$)
                  </label>
                  <input
                    type="number"
                    step="0.01"
                    value={selectedProductionCell.value || ''}
                    onChange={(e) => setSelectedProductionCell({
                      ...selectedProductionCell,
                      value: parseFloat(e.target.value) || 0
                    })}
                    className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500"
                  />
                </div>

                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Trabalhou em outra equipe?
                  </label>
                  <select
                    value={selectedProductionCell.workedTeam || ''}
                    onChange={(e) => setSelectedProductionCell({
                      ...selectedProductionCell,
                      workedTeam: e.target.value || null
                    })}
                    className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500"
                  >
                    <option value="">Não (equipe padrão: {selectedProductionCell.employee.team})</option>
                    {teams.filter(t => t !== selectedProductionCell.employee.team).map(team => (
                      <option key={team} value={team}>{team}</option>
                    ))}
                  </select>
                  {selectedProductionCell.workedTeam && (
                    <p className="text-xs text-orange-600 mt-1">
                      ⚠️ Este funcionário trabalhou na {selectedProductionCell.workedTeam} neste dia
                    </p>
                  )}
                </div>

                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Observações
                  </label>
                  <textarea
                    value={selectedProductionCell.observation || ''}
                    onChange={(e) => setSelectedProductionCell({
                      ...selectedProductionCell,
                      observation: e.target.value
                    })}
                    className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500"
                    rows="3"
                    placeholder="Ex: Substituição, hora extra, trabalho especial..."
                  />
                </div>
              </div>

              <div className="flex gap-3 mt-6">
                <button
                  onClick={saveProductionDetails}
                  className="flex-1 px-6 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors"
                >
                  Salvar
                </button>
                <button
                  onClick={() => {
                    setShowProductionModal(false);
                    setSelectedProductionCell(null);
                  }}
                  className="flex-1 px-6 py-2 bg-gray-300 text-gray-700 rounded-lg hover:bg-gray-400 transition-colors"
                >
                  Cancelar
                </button>
              </div>
            </div>
          </div>
        )}

        {activeTab === 'report' && (
          <div className="space-y-6">
            <div className="flex justify-end gap-2 mb-4 no-print">
              <button
                onClick={handlePrint}
                className="flex items-center gap-2 bg-gray-600 text-white px-4 py-2 rounded-lg hover:bg-gray-700 transition-colors"
              >
                <Printer size={20} />
                Imprimir
              </button>
              <button
                onClick={exportReportToPDF}
                className="flex items-center gap-2 bg-green-600 text-white px-4 py-2 rounded-lg hover:bg-green-700 transition-colors"
              >
                <Download size={20} />
                Exportar PDF
              </button>
            </div>

            <div className="bg-white rounded-lg shadow-lg p-6">
              <h2 className="text-xl font-bold text-gray-800 mb-6">Relatório Completo - Todos os Funcionários</h2>
              
              {employees.length === 0 ? (
                <div className="text-center py-12 text-gray-500">
                  Nenhum funcionário cadastrado ainda.
                </div>
              ) : (
                <div className="space-y-6">
                  {teams.map(team => {
                    const teamEmps = employees.filter(e => e.team === team);
                    if (teamEmps.length === 0) return null;
                    
                    const teamTotal = teamEmps.reduce((sum, emp) => sum + calculateEmployeeTotal(emp.id), 0);
                    const maxValue = Math.max(...teamEmps.map(emp => calculateEmployeeTotal(emp.id)));
                    
                    return (
                      <div key={team} className="border-2 border-gray-200 rounded-lg p-5">
                        <div className="flex justify-between items-center mb-4 pb-3 border-b-2 border-gray-300">
                          <h3 className="text-lg font-bold text-gray-800">{team}</h3>
                          <div className="text-right">
                            <p className="text-sm text-gray-600">Total da Equipe</p>
                            <p className="text-xl font-bold text-green-600">R$ {teamTotal.toFixed(2)}</p>
                          </div>
                        </div>
                        
                        <div className="space-y-3">
                          {teamEmps
                            .sort((a, b) => calculateEmployeeTotal(b.id) - calculateEmployeeTotal(a.id))
                            .map(emp => {
                              const total = calculateEmployeeTotal(emp.id);
                              const percentage = maxValue > 0 ? (total / maxValue) * 100 : 0;
                              
                              return (
                                <div key={emp.id} className="bg-gray-50 rounded-lg p-4">
                                  <div className="flex justify-between items-start mb-2">
                                    <div className="flex-1">
                                      <p className="font-semibold text-gray-800">{emp.name}</p>
                                      <p className="text-sm text-gray-600">
                                        {emp.position} • R$ {emp.dailyValue.toFixed(2)}/dia
                                        {emp.badge && <span className="ml-2">• Ficha: {emp.badge}</span>}
                                      </p>
                                      {productions.some(p => p.employeeId === emp.id && p.workedTeam) && (
                                        <p className="text-xs text-orange-600 mt-1">
                                          ⚠️ Trabalhou em outras equipes em alguns dias
                                        </p>
                                      )}
                                    </div>
                                    <div className="text-right ml-4">
                                      <p className="text-lg font-bold text-green-600">R$ {total.toFixed(2)}</p>
                                      <p className="text-xs text-gray-500">
                                        {productions.filter(p => p.employeeId === emp.id && p.value > 0).length} dias trabalhados
                                      </p>
                                    </div>
                                  </div>
                                  
                                  <div className="mt-3">
                                    <div className="w-full bg-gray-200 rounded-full h-3 overflow-hidden">
                                      <div
                                        className="bg-gradient-to-r from-blue-500 to-green-500 h-3 rounded-full transition-all duration-500"
                                        style={{ width: `${percentage}%` }}
                                      ></div>
                                    </div>
                                  </div>
                                </div>
                              );
                            })}
                        </div>
                      </div>
                    );
                  })}
                </div>
              )}
            </div>

            <div className="bg-white rounded-lg shadow-lg p-6">
              <h2 className="text-xl font-bold text-gray-800 mb-4">Gráfico Comparativo por Equipe</h2>
              <div className="space-y-4">
                {teams.map(team => {
                  const teamEmps = employees.filter(e => e.team === team);
                  const teamTotal = teamEmps.reduce((sum, emp) => sum + calculateEmployeeTotal(emp.id), 0);
                  const maxTeamValue = Math.max(...teams.map(t => 
                    employees.filter(e => e.team === t).reduce((sum, emp) => sum + calculateEmployeeTotal(emp.id), 0)
                  ));
                  const percentage = maxTeamValue > 0 ? (teamTotal / maxTeamValue) * 100 : 0;
                  
                  return (
                    <div key={team}>
                      <div className="flex justify-between items-center mb-2">
                        <span className="font-semibold text-gray-700">{team}</span>
                        <span className="font-bold text-green-600">R$ {teamTotal.toFixed(2)}</span>
                      </div>
                      <div className="w-full bg-gray-200 rounded-full h-8 overflow-hidden">
                        <div
                          className="bg-gradient-to-r from-indigo-500 to-purple-500 h-8 rounded-full flex items-center justify-end pr-3 transition-all duration-500"
                          style={{ width: `${percentage}%` }}
                        >
                          {percentage > 15 && (
                            <span className="text-white text-sm font-medium">
                              {teamEmps.length} funcionários
                            </span>
                          )}
                        </div>
                      </div>
                    </div>
                  );
                })}
              </div>
              
              <div className="mt-6 pt-6 border-t-2">
                <div className="flex justify-between items-center">
                  <span className="text-lg font-bold text-gray-800">TOTAL GERAL</span>
                  <span className="text-2xl font-bold text-green-600">
                    R$ {employees.reduce((sum, emp) => sum + calculateEmployeeTotal(emp.id), 0).toFixed(2)}
                  </span>
                </div>
                <p className="text-sm text-gray-600 mt-2">
                  {employees.length} funcionários • {teams.filter(t => employees.some(e => e.team === t)).length} equipes ativas
                </p>
              </div>
            </div>
          </div>
        )}
      </div>
    </div>
  );
};

const EmployeeForm = ({ positions, teams, onSave, onCancel, initialData }) => {
  const [formData, setFormData] = useState(initialData || {
    name: '',
    badge: '',
    position: positions[0].name,
    team: teams[0],
    dailyValue: positions[0].baseValue
  });

  const handlePositionChange = (position) => {
    const positionData = positions.find(p => p.name === position);
    setFormData({
      ...formData,
      position,
      dailyValue: positionData.baseValue
    });
  };

  const handleSubmit = () => {
    if (!formData.name.trim()) {
      alert('Por favor, preencha o nome do funcionário');
      return;
    }
    onSave(formData);
  };

  return (
    <div className="bg-blue-50 p-6 rounded-lg mb-6">
      <h3 className="text-lg font-bold text-gray-800 mb-4">
        {initialData ? 'Editar Funcionário' : 'Novo Funcionário'}
      </h3>
      <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
        <div>
          <label className="block text-sm font-medium text-gray-700 mb-1">Nome Completo *</label>
          <input
            type="text"
            value={formData.name}
            onChange={(e) => setFormData({ ...formData, name: e.target.value })}
            className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
          />
        </div>
        <div>
          <label className="block text-sm font-medium text-gray-700 mb-1">N. Ficha</label>
          <input
            type="text"
            value={formData.badge}
            onChange={(e) => setFormData({ ...formData, badge: e.target.value })}
            className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
          />
        </div>
        <div>
          <label className="block text-sm font-medium text-gray-700 mb-1">Cargo *</label>
          <select
            value={formData.position}
            onChange={(e) => handlePositionChange(e.target.value)}
            className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
          >
            {positions.map(pos => (
              <option key={pos.name} value={pos.name}>{pos.name}</option>
            ))}
          </select>
        </div>
        <div>
          <label className="block text-sm font-medium text-gray-700 mb-1">Equipe *</label>
          <select
            value={formData.team}
            onChange={(e) => setFormData({ ...formData, team: e.target.value })}
            className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
          >
            {teams.map(team => (
              <option key={team} value={team}>{team}</option>
            ))}
          </select>
        </div>
        <div>
          <label className="block text-sm font-medium text-gray-700 mb-1">Valor Diário (R$) *</label>
          <input
            type="number"
            step="0.01"
            value={formData.dailyValue}
            onChange={(e) => setFormData({ ...formData, dailyValue: parseFloat(e.target.value) || 0 })}
            className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
          />
        </div>
      </div>
      <div className="flex gap-3 mt-6">
        <button
          onClick={handleSubmit}
          className="px-6 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors"
        >
          {initialData ? 'Salvar Alterações' : 'Adicionar Funcionário'}
        </button>
        <button
          onClick={onCancel}
          className="px-6 py-2 bg-gray-300 text-gray-700 rounded-lg hover:bg-gray-400 transition-colors"
        >
          Cancelar
        </button>
      </div>
    </div>
  );
};

export default BonusManagerApp;
