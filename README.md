# Automated-GST-Invoicing-System-Development-using-Firebase-Firestore-and-Cloud-Functions.
A comprehensive guide on the system design, including an explanation of how IGST and SGST/CGST calculations are handled

exports.processBooking = functions.firestore
  .document('bookings/{bookingId}')
  .onUpdate((change, context) => {
    const newData = change.after.data();
    const oldData = change.before.data();

    if (newData.status === 'finished' && oldData.status !== 'finished') {

      generateInvoice(newData);
    }

    return null;
  });

function generateInvoice(data) {
  const totalBookingAmount = data.totalBookingAmount;
  
  
  const gstRate = 18; // Ex GST slab rate (18%)
  const gstAmount = (totalBookingAmount * gstRate) / 100;
  
 
  const igst = gstAmount / 2;
  const sgstCgst = gstAmount / 2;
  
  
  const invoice = {
    name: data.name,
    totalBookingAmount: totalBookingAmount,
    gstAmount: gstAmount,
    igst: igst,
    sgstCgst: sgstCgst,
  };
  
  
  integrateWithGSTAPI(invoice);
}
function integrateWithGSTAPI(invoice) {
  const gstAPIEndpoint = 'https://example.com/gst-api';
  
  const options = {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(invoice),
  };
  
  fetch(gstAPIEndpoint, options)
    .then(response => {
      if (!response.ok) {
        throw new Error('Failed to file GST');
      }
      return response.json();
    })
    .then(data => {
      console.log('GST filed successfully:', data);
    })
    .catch(error => {
      console.error('Error filing GST:', error);
    });
}
